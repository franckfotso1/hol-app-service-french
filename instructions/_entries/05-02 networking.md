---
sectionid: lab4-network
sectionclass: h2
title: Vnet integration
parent-id: lab-4
---


vous allez configurer une application App Service avec une communication sécurisée, isolée du réseau, aux services principaux. Lorsque vous avez terminé, vous disposerez d’une application App Service qui accède à la fois à Key Vault et à Cognitive Services via un réseau virtuel Azure, et aucun autre trafic n’est autorisé à accéder à ces ressources principales. Tout le trafic sera isolé au sein de votre réseau virtuel à l'aide de l’intégration du réseau virtuel et de points de terminaison privés.

Pour le moment, les secrets de connexion sont stockés sous forme de paramètres d’application dans l’application App Service. Cette approche sécurise déjà les secrets de connexion du codebase de votre application. Toutefois, un contributeur autorisé à gérer votre application peut également voir les paramètres de l’application. Au cours de cette étape, vous allez déplacer les secrets de connexion dans un coffre de clés et verrouiller l’accès afin d’être la seule personne à pouvoir le gérer. Seule l’application App Service pourra le lire à l’aide de son identité managée.
À présent, définissez les paramètres d’application comme références du coffre de clés.

#### Créez un [groupe de ressources](https://learn.microsoft.com/fr-fr/azure/azure-resource-manager/management/manage-resource-groups-cli) relatif à votre application

{% collapsible %}

```bash
RESOURCE_GROUP="<your-resource-group-name>"
LOCATION="<your-region>"
az group create --name $RESOURCE_GROUP --location "$LOCATION"
```

{% endcollapsible %}

#### déployez cette application .NET 7 sur un plan App Service Windows Standard

{% collapsible %}

```bash
APP_SERVICE_PLAN="<your-app-service-plan-name>"
# Créez un plan App Service Standard avec 4 instances de machine Windows
az appservice plan create -g $RESOURCE_GROUP -n $APP_SERVICE_PLAN --is-linux --number-of-workers 4 --sku S1
```

```bash
APP_NAME="<your-dotnet-web-app-name>"
# Obtenez la liste des runtimes supportés pour chaque OS
az webapp list-runtimes
# Créez la web app
az webapp create -g $RESOURCE_GROUP -n $APP_NAME -p  $APP_SERVICE_PLAN -r "dotnet:7" 
```

```bash
GIT_REPO="https://github.com/Azure-Samples/app-service-language-detector.git"
az webapp deployment source config --name $APP_NAME --resource-group $RESOURCE_GROUP `
--repo-url $GIT_REPO --branch master --manual-integration
```

```bash
az webapp show -n $APP_NAME -g $RESOURCE_GROUP --query "defaultHostName"
```

{% endcollapsible %}

#### Créez une ressource Cognitive Services

{% collapsible %}

```bash
CS_ACCOUNT_NAME="<your-cognitive-services-account-name>"
az cognitiveservices account create -g $RESOURCE_GROUP -n $CS_ACCOUNT_NAME -l "$LOCATION" --kind TextAnalytics --sku F0 --custom-domain $CS_ACCOUNT_NAME
```

{% endcollapsible %}

#### Créez le key Vault qui contiendra les secrets de notre app

{% collapsible %}

```bash
VAULT_NAME="<vault-name>"
az keyvault create -g $RESOURCE_GROUP -n $VAULT_NAME -l "$LOCATION" --sku standard --enable-rbac-authorization
```

{% endcollapsible %}

#### Sécurisez le key Vault et donnez-vous le rôle RBAC `Key Vault Secrets Officer`

{% collapsible %}

```bash
VAULT_ID=$(az keyvault show --name $VAULT_NAME --query id --output tsv)
MY_ID=$(az ad signed-in-user show --query id --output tsv)

az role assignment create --role "Key Vault Secrets Officer" --assignee-object-id $MY_ID --assignee-principal-type User --scope $VAULT_ID
```

{% endcollapsible %}

#### Activez l’identité managée affectée par le système de votre application et donnez-lui le rôle RBAC `Key Vault Secrets User`

{% collapsible %}

```bash
az webapp identity assign -g $RESOURCE_GROUP -n $APP_NAME --scope $VAULT_ID --role  "Key Vault Secrets User"
```

{% endcollapsible %}

#### Récupérez la clé d'abonnement de Cognitive Services

{% collapsible %}

```bash
$CS_ACCOUNT_KEY = az cognitiveservices account keys list -g $RESOURCE_GROUP -n $CS_ACCOUNT_NAME --query key1 --output tsv
```

{% endcollapsible %}

#### Ajoutez le nom de la ressource Cognitive Services et la clé d’abonnement aux secrets du coffre, puis enregistrez leur ID comme variables d’environnement pour l’étape suivante

{% collapsible %}

```bash
$CS_NAME_KVUri = az keyvault secret set --vault-name $VAULT_NAME --name cs_name --value $CS_ACCOUNT_NAME --query id --output tsv
$CS_KEY_KVUri = az keyvault secret set --vault-name $VAULT_NAME --name cs_key --value $CS_ACCOUNT_KEY --query id --output tsv
```

{% endcollapsible %}

#### Enfin, définir les secrets sous la forme des paramètres d’application et les valeurs comme références  au keyVault

{% collapsible %}

```bash
az webapp config appsettings set -g $RESOURCE_GROUP - $APP_NAME --settings CS_ACCOUNT_NAME="@Microsoft.KeyVault(SecretUri=$CS_NAME_KVUri)" CS_ACCOUNT_KEY="@Microsoft.KeyVault(SecretUri=$CS_KEY_KVUri)"
```

{% endcollapsible %}

> Votre application se connecte maintenant à Cognitive Services avec des secrets conservés dans votre coffre de clés, sans aucune modification du code

A présent isolons le traffic réseau pour que la Web App accède à d'autres services via un réseau virtuel.

#### Créez un réseau virtuel

{% collapsible %}

```bash
VNET_NAME="<virtual-network-name>"
az network vnet create -g $RESOURCE_GROUP -l "$LOCATION" --name $VNET_NAME --address-prefixes 10.0.0.0/16
```

{% endcollapsible %}

#### Créez un sous-réseau pour l’intégration de réseau virtuel App Service

{% collapsible %}

```bash
VNET_NAME="<virtual-network-name>"
az network vnet subnet create -g $RESOURCE_GROUP --vnet-name $VNET_NAME --name vnet-integration-subnet --address-prefixes 10.0.0.0/24 --delegations Microsoft.Web/serverfarms --disable-private-endpoint-network-policies false
```

{% endcollapsible %}

#### Créez un autre sous-réseau pour les points de terminaison privés.

{% collapsible %}

```bash
## Pour les sous-réseaux du point de terminaison privé, vous devez désactiver les stratégies réseau des points de terminaison privés.
az network vnet subnet create -g $RESOURCE_GROUP --vnet-name $VNET_NAME --name private-endpoint-subnet --address-prefixes 10.0.1.0/24 --disable-private-endpoint-network-policies true
```

{% endcollapsible %}

#### Créez deux zones DNS privées, une pour votre ressource Cognitive Services et une pour votre coffre de clés

{% collapsible %}

```bash
az network private-dns zone create -g $RESOURCE_GROUP --name privatelink.cognitiveservices.azure.com
az network private-dns zone create -g $RESOURCE_GROUP --name privatelink.vaultcore.azure.net
```

{% endcollapsible %}

#### Liez les zones DNS privées au réseau virtuel

{% collapsible %}

```bash
az network private-dns link vnet create -g $RESOURCE_GROUP --name cognitiveservices-zonelink --zone-name privatelink.cognitiveservices.azure.com --virtual-network $VNET_NAME --registration-enabled False

az network private-dns link vnet create -g $RESOURCE_GROUP --name vaultcore-zonelink --zone-name privatelink.vaultcore.azure.net --virtual-network $VNET_NAME --registration-enabled False
```

{% endcollapsible %}

#### Créer des points de terminaison privés

##### Dans le sous-réseau de terminaux privés de votre réseau virtuel, créez un terminal privé pour votre Cognitive Service.

{% collapsible %}

```bash
# Get Cognitive Services resource ID
CS_ID=$(az cognitiveservices account show -g $RESOURCE_GROUP --name $CS_ACCOUNT_NAME --query id --output tsv)

az network private-endpoint create -g $RESOURCE_GROUP --name securecstext-pe -l "$LOCATION" --connection-name securecstext-pc --private-connection-resource-id $CS_ID --group-id account --vnet-name  $VNET_NAME --subnet private-endpoint-subnet
```

{% endcollapsible %}

##### Créez un groupe de zones DNS pour le point de terminaison privé Cognitive Services

{% collapsible %}

```bash
az network private-endpoint dns-zone-group create -g $RESOURCE_GROUP --endpoint-name securecstext-pe --name securecstext-zg --private-dns-zone privatelink.cognitiveservices.azure.com --zone-name privatelink.cognitiveservices.azure.com
```

{% endcollapsible %}

##### Bloquez le trafic public vers la ressource Cognitive Services.

{% collapsible %}

```bash
az rest --uri $CS_ID?api-version=2021-04-30 --method PATCH --body '{"properties":{"publicNetworkAccess":"Disabled"}}' --headers 'Content-Type=application/json'

# Repeat following command until output is "Succeeded"
az cognitiveservices account show -g $RESOURCE_GROUP --name $CS_ACCOUNT_NAME --query properties.provisioningState
```

{% endcollapsible %}

> Vous pouvez toujours charger l’application, mais si vous essayez de cliquer sur le bouton Détecter, vous recevez une erreur HTTP 500. L’application a perdu sa connectivité à la ressource Cognitive Services via la mise en réseau partagée

##### Répétez les étapes ci-dessus pour le Key Vault.

{% collapsible %}

```bash
# Create private endpoint for key vault
VAULT_ID=$(az keyvault show --name $VAULT_NAME --query id --output tsv)
az network private-endpoint create -g $RESOURCE_GROUP --name securekeyvault-pe -l "$LOCATION" --connection-name securekeyvault-pc --private-connection-resource-id $VAULT_ID --group-id vault --vnet-name $VNET_NAME --subnet private-endpoint-subnet
# Create DNS zone group for the endpoint
az network private-endpoint dns-zone-group create -g $RESOURCE_GROUP --endpoint-name securekeyvault-pe --name securekeyvault-zg --private-dns-zone privatelink.vaultcore.azure.net --zone-name privatelink.vaultcore.azure.net
# Block public traffic to key vault
az keyvault update --name $VAULT_NAME --default-action Deny
```

{% endcollapsible %}

##### Renitialisez les paramètres de l’application

{% collapsible %}

```bash
az webapp config appsettings set -g $RESOURCE_GROUP --name $APP_NAME --settings CS_ACCOUNT_NAME="@Microsoft.KeyVault(SecretUri=$CS_NAME_KVUri)" CS_ACCOUNT_KEY="@Microsoft.KeyVault(SecretUri=$CS_KEY_KVUri)"
```

{% endcollapsible %}

> Vous ne pouvez plus charger l’application, car elle ne peut plus accéder aux références du coffre de clés. L’application a perdu sa connectivité au coffre de clés par le biais de la mise en réseau partagée

#### Activez l’intégration du réseau virtuel sur votre application

{% collapsible %}

```bash
az webapp vnet-integration add -g $RESOURCE_GROUP --name $APP_NAME --vnet $VNET_NAME --subnet vnet-integration-subnet
# appliquez le protocole HTTPS pour les demandes entrantes
az webapp update -g $RESOURCE_GROUP --name $APP_NAME --https-only
```

{% endcollapsible %}

> L’intégration du réseau virtuel permet au trafic sortant de circuler directement dans le réseau virtuel
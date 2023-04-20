---
sectionid: lab4-network
sectionclass: h2
title: Vnet integration
parent-id: lab-4
---


A présent isolons le trafic pour que la Web App accède à d'autres services via un réseau virtuel.

#### Créez un réseau virtuel (VNet)

{% collapsible %}

```bash
VNET_NAME="<virtual-network-name>"
az network vnet create -g $RESOURCE_GROUP -l "$LOCATION" --name $VNET_NAME --address-prefixes 10.0.0.0/16
```

{% endcollapsible %}

#### Créez un sous-réseau (Subnet) pour l’intégration VNet à App Service

{% collapsible %}

```bash
VNET_NAME="<virtual-network-name>"
az network vnet subnet create -g $RESOURCE_GROUP --vnet-name $VNET_NAME --name vnet-integration-subnet --address-prefixes 10.0.0.0/24 --delegations Microsoft.Web/serverfarms
```

{% endcollapsible %}

#### Créez un autre sous-réseau (Subnet) pour les [private endpoints](https://learn.microsoft.com/fr-fr/azure/private-link/private-endpoint-overview)

{% collapsible %}

```bash
## Pour les sous-réseaux du point de terminaison privé, vous devez désactiver les stratégies réseau des points de terminaison privés.
az network vnet subnet create -g $RESOURCE_GROUP --vnet-name $VNET_NAME --name private-endpoint-subnet --address-prefixes 10.0.1.0/24 --disable-private-endpoint-network-policies true
```

{% endcollapsible %}

#### Créez deux zones DNS privées, une pour votre ressource Cognitive Services et une pour le Key Vault

{% collapsible %}

```bash
az network private-dns zone create -g $RESOURCE_GROUP --name privatelink.cognitiveservices.azure.com
az network private-dns zone create -g $RESOURCE_GROUP --name privatelink.vaultcore.azure.net
```

{% endcollapsible %}

#### Liez les zones DNS privées au VNet

{% collapsible %}

```bash
az network private-dns link vnet create -g $RESOURCE_GROUP --name cognitiveservices-zonelink --zone-name privatelink.cognitiveservices.azure.com --virtual-network $VNET_NAME --registration-enabled False

az network private-dns link vnet create -g $RESOURCE_GROUP --name vaultcore-zonelink --zone-name privatelink.vaultcore.azure.net --virtual-network $VNET_NAME --registration-enabled False
```

{% endcollapsible %}

#### Créer des points de terminaison privés

##### Créez un [private endpoint](https://learn.microsoft.com/en-us/cli/azure/network/private-endpoint?view=azure-cli-latest) pour votre Cognitive Service dans le subnet correspondant

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

#### Activez l’intégration du VNet à votre application

{% collapsible %}

```bash
az webapp vnet-integration add -g $RESOURCE_GROUP --name $APP_NAME --vnet $VNET_NAME --subnet vnet-integration-subnet
# appliquez le protocole HTTPS pour les demandes entrantes
az webapp update -g $RESOURCE_GROUP --name $APP_NAME --https-only
```

{% endcollapsible %}

> Cet intégration permet au trafic sortant de circuler directement dans le VNet

-------------------------------

#### Créez le [key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-cli) qui contiendra les secrets de notre app

{% collapsible %}

```bash
VAULT_NAME="<vault-name>"
az keyvault create -g $RESOURCE_GROUP -n $VAULT_NAME -l "$LOCATION" --sku standard --enable-rbac-authorization
```

{% endcollapsible %}

#### Donnez-vous le rôle RBAC `Key Vault Secrets Officer` sur le Key Vault

{% collapsible %}

```bash
VAULT_ID=$(az keyvault show --name $VAULT_NAME --query id --output tsv)
MY_ID=$(az ad signed-in-user show --query id --output tsv)

az role assignment create --role "Key Vault Secrets Officer" --assignee-object-id $MY_ID --assignee-principal-type User --scope $VAULT_ID
```

{% endcollapsible %}

#### Affectez l’identité managée d'Azure Active Directory à votre application et donnez-lui le rôle RBAC `Key Vault Secrets User`

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

> Votre application se connecte maintenant à Cognitive Services avec des secrets conservés dans votre Key Vault, sans aucune modification du code.

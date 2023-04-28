---
sectionid: lab4-key
sectionclass: h2
title: Key vault
parent-id: lab-4
---


vous allez déplacer les secrets de connexion de la BD dans le Key Vault et verrouiller l’accès afin d’être la seule personne à pouvoir le gérer. Seule l’application App Service pourra le lire à l’aide de son identité managée.

#### Créez le [key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-cli)

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

#### Récupérez la connection string de votre base de données

{% collapsible %}

```bash
$COSMOSDB_KEY = az cosmosdb keys list --name $COSMOSDB_ACCOUNT --resource-group $RESOURCE_GROUP --type connection-strings --query connectionStrings[0].connectionString --output tsv
```

{% endcollapsible %}

#### Ajoutez le nom de la base de données et la connection String aux secrets du coffre, puis enregistrez leur ID comme variables d’environnement pour l’étape suivante

{% collapsible %}

```bash
$CS_NAME_KVUri = az keyvault secret set --vault-name $VAULT_NAME --name cs_name --value $DATABASE_NAME --query id --output tsv
$CS_KEY_KVUri = az keyvault secret set --vault-name $VAULT_NAME --name cs_key --value $COSMOSDB_KEY --query id --output tsv
```

{% endcollapsible %}

#### Enfin, Renitialisez les paramètres de l’application en prenant comme valeurs les références  au keyVault

{% collapsible %}

```bash
az webapp config appsettings set -g $RESOURCE_GROUP - $APP_NAME --settings CS_ACCOUNT_NAME="@Microsoft.KeyVault(SecretUri=$CS_NAME_KVUri)" CS_ACCOUNT_KEY="@Microsoft.KeyVault(SecretUri=$CS_KEY_KVUri)"
```

{% endcollapsible %}

> Votre application se connecte maintenant à CosmosDB avec des secrets conservés dans votre Key Vault, sans aucune modification du code.

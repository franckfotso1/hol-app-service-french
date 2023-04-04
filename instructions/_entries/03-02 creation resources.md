---
sectionid: lab2-resources-creation
sectionclass: h2
title: Creation ressources
parent-id: lab-2
---

- Fork puis clone les sources du projet [ici](https://github.com/Azure-Samples/msdocs-nodejs-mongodb-azure-sample-app)
  
#### Créez un [groupe de ressources](https://learn.microsoft.com/fr-fr/azure/azure-resource-manager/management/manage-resource-groups-cli) pour le Lab 2

La convention de nommage pour le groupes de ressources sera la suivante: `rg-<environment>-<region>-<application-name>-<owner>-<instance>`

{% collapsible %}

```bash
RESOURCE_GROUP="<your-resource-group-name>"
LOCATION="<your-region>"
az group create --name $RESOURCE_GROUP --location "$LOCATION"
```

{% endcollapsible %}

#### Créez un plan AppService Linux avec un tier Standard (minimum requis pour les Slots)

La convention de nommage pour le plan App Service sera la suivante: `aps-<environment>-<region>-<application-name>-<owner>-<instance>`

{% collapsible %}

```bash
APP_SERVICE_PLAN="<your-app-service-plan-name>"
# Créez un plan App Service Standard avec 2 instances de machine Linux
az appservice plan create -g $RESOURCE_GROUP -n $APP_SERVICE_PLAN --is-linux --number-of-workers 2 --sku S1
```

{% endcollapsible %}

#### Créez une Web App sur ce plan App Service en spécifiant le runtime NODE 18

La convention de nommage pour la web app sera la suivante: `app-<environment>-<region>-<application-name>-<app-suffix><owner>-<instance>`

{% collapsible %}

```bash
APP_NAME="<your-express-web-app-name>"
# Obtenez la liste des runtimes supportés pour chaque OS
az webapp list-runtimes
# Créez la web app
az webapp create -g $RESOURCE_GROUP -n $APP_NAME -p $APP_SERVICE_PLAN -r "NODE:18-lts" 
```

{% endcollapsible %}

##### Créez une base de données Azure Cosmos DB for MongoDB

> C'est une base de données native Cloud qui propose une API 100 % compatible avec MongoDB.

La convention de nommage pour le compte Cosmos DB pour MONGODB est la suivante: `cosmon-<environment>-<region>-<application-name>-<app-suffix><owner>-<instance>` et pour la base de données : `cosmos-<environment>-<region>-<application-name>-<app-suffix><owner>-<instance>`

{% collapsible %}

```bash
COSMOSDB_ACCOUNT="<cosmos-db-account-name>"
# Créez un nouveau compte Cosmos DB avec l'API MongoDB
az cosmosdb create --name $COSMOSDB_ACCOUNT --kind MongoDB -g $RESOURCE_GROUP
```

```bash
DATABASE_NAME="<mongo-db-database-name>"
# Créez une nouvelle base de données MongoDB
az cosmosdb mongodb database create --account-name $COSMOSDB_ACCOUNT -g $RESOURCE_GROUP --name $DATABASE_NAME
```

{% endcollapsible %}

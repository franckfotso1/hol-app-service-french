---
sectionid: first-app
sectionclass: h2
title: Une première application
parent-id: lab-1
---


### Diagramme de l'architecture

{% collapsible %}
![Première app](/media/lab1/lab_1_arch.png)
{% endcollapsible %}

Examinons quelques paramètres dont vous avez besoin pour créer une application App Service :

- **le nom** : le nom de l’application doit être globalement unique
- **le format de publication** : App Service publie votre application sous forme de code ou conteneur Docker
- **la pile d'exécution** : langage, version du SDK
- **le système d’exploitation** : Linux ou Windows
- **le plan App Service** : l'application doit être associée à un plan App Service pour établir les ressources et les capacités disponibles.

> D’autres applications peuvent également être associées au même plan App Service et donc utiliser les mêmes ressources.

#### Créez un [groupe de ressources](https://learn.microsoft.com/fr-fr/azure/azure-resource-manager/management/manage-resource-groups-cli) relatif à votre application

N’oubliez pas que la convention de nommage pour le groupes de ressources sera la suivante: `rg-<environment>-<region>-<application-name>-<owner>-<instance>`

{% collapsible %}

```bash
RESOURCE_GROUP="<your-resource-group-name>"
LOCATION="<your-region>"
az group create --name $RESOURCE_GROUP --location "$LOCATION"
```

{% endcollapsible %}

#### Créez un [plan AppService Linux](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans) avec un tier Standard

N’oubliez pas que la convention de nommage pour le plan App Service sera la suivante: `aps-<environment>-<region>-<application-name>-<owner>-<instance>`

{% collapsible %}

```bash
APP_SERVICE_PLAN="<your-app-service-plan-name>"
# Créez un plan App Service Standard avec 4 instances de machine Linux
az appservice plan create -g $RESOURCE_GROUP -n $APP_SERVICE_PLAN --is-linux --number-of-workers 4 --sku S1
```

{% endcollapsible %}

> le tier d'un plan App Service détermine les fonctionnalités App Service que vous obtenez et combien vous payez pour le plan.

{% collapsible %}

![ASP tier ](/media/lab1/tier_asp.png)

{% endcollapsible %}

#### Lancer l'application php en local (facultatif)

```bash
# clonez le dépôt
git clone https://github.com/Azure-Samples/php-docs-hello-world
cd php-docs-hello-world
# Démarrez l'application localement
php -S localhost:8080
curl http://localhost:8080
```

#### Créez une [Web App](https://learn.microsoft.com/en-us/azure/app-service/scripts/cli-deploy-github)sur ce plan App Service Linux en spécifiant le runtime PHP 8.0

N’oubliez pas que la convention de nommage pour la web app sera la suivante: `app-<environment>-<region>-<application-name>-<app-suffix><owner>-<instance>`

{% collapsible %}

```bash
APP_NAME="<your-php-web-app-name>"
# Obtenez la liste des runtimes supportés pour chaque OS
az webapp list-runtimes
# Créez la web app
az webapp create -g $RESOURCE_GROUP -n $APP_NAME -p  $APP_SERVICE_PLAN -r "PHP:8.0" 
```

{% endcollapsible %}

> un nom de domaine unique sous la forme **AppName.azurewebsites.net** sera attribuée à la web app

{% collapsible %}
![App UI default](/media/lab1/php_quick_app.png)
{% endcollapsible %}

#### [Déployez](https://learn.microsoft.com/en-us/azure/app-service/scripts/cli-deploy-github) manuellement le code de l'application php depuis le repo Github

{% collapsible %}

```bash
GIT_REPO="https://github.com/Azure-Samples/php-docs-hello-world"
az webapp deployment source config --name $APP_NAME --resource-group $RESOURCE_GROUP `
--repo-url $GIT_REPO --branch master --manual-integration
```

{% endcollapsible %}

Une fois le déploiment effectué, Sélectionnez **Accéder à la ressource**. Pour avoir un apercu de l'application web, cliquez sur l'URL en haut à droite du portail ou celui renvoyé par la commande suivante :

```bash
az webapp show -n $APP_NAME -g $RESOURCE_GROUP --query "defaultHostName"
```

{% collapsible %}
![App overview](/media/lab1/view_php_app.png)
![App UI default](/media/lab1/app_deploy.png)
{% endcollapsible %}

#### Créez l'application web ASP.NET Core

- Ouvrez Visual Studio 2022, puis sélectionnez Créer un projet
  
{% collapsible %}
![new project](/media/lab1/create_new_project.png)
{% endcollapsible %}

- Dans Créer un projet, recherchez et sélectionnez Application web ASP.NET Core, puis sélectionnez Suivant.
{% collapsible %}
![asp web app](/media/lab1/asp_web_app.png)
{% endcollapsible %}

- Dans Configurer votre nouveau projet, nommez l’application app-dev-we-hol-ms-02, puis sélectionnez Suivant
  
- Sélectionnez .NET Core 6.0 (prise en charge à long terme).
{% collapsible %}
![choose stack](/media/lab1/stack_asp.png)
{% endcollapsible %}

- Assurez-vous que Type d’authentification est défini sur Aucun et Sélectionnez Create (Créer)
  
#### Publiez l'application sur le meme plan App Service Linux

- cliquez avec le bouton droit sur le projet app-dev-we-hol-ms-02, puis sélectionnez Publier.
  
{% collapsible %}
![publish](/media/lab1/asp_publish.png)
{% endcollapsible %}

- Dans Publier, sélectionnez Azure, puis Suivant
 {% collapsible %}
![azure](/media/lab1/asp_azure.png)
{% endcollapsible %}
  
- Choisissez la cible spécifique, Azure App Service (Linux)

- Sélectionnez **Ajouter un compte ou Connexion** pour vous connecter à votre abonnement Azure
  
- À droite d’Instances App Service, sélectionnez + et choisir le nom, l'ASP et le RG puis sélectionnez Créer
{% collapsible %}
![asp-deploy](/media/lab1/dotnet_app_deploy.png)
{% endcollapsible %}

- Dans la page Publier, sélectionnez Publier. Si vous voyez un message d’avertissement, sélectionnez Continuer.

> Visual Studio build, génère un profil de publication et publie l’application web ASP.NET Core 6.0 sur Azure, puis la démarre dans le navigateur par défaut

#### Vérifiez la présence des deux Web Apps dans votre plan App Service Linux

{% collapsible %}

- via CLI

```bash
az webapp list -g $RESOURCE_GROUP --output table
```

- via Portail
  ![asp-deploy](/media/lab1/asp_list_apps.png)

{% endcollapsible %}

#### Supprimez le groupe de ressources

{% collapsible %}

```bash
az group delete -n $RESOURCE_GROUP
```

{% endcollapsible %}

> Une bonne pratique consiste évidemment à automatiser le déploiement de son infrastructure à l'aide doutils Iac comme [Bicep](https://learn.microsoft.com/fr-fr/azure/app-service/provision-resource-bicep) ou [Terraform](https://learn.microsoft.com/fr-fr/azure/app-service/provision-resource-terraform).

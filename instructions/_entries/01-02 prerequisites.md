---
sectionid: prereq
sectionclass: h2
title: Prérequis
parent-id: intro
---

### Prérequis

Ce workshop va demander les éléments suivants:

- une souscription Azure
- Azure CLI
- [VS Code](https://code.visualstudio.com/) ou équivalent
- [Visual Studio 2022](https://visualstudio.microsoft.com/fr/vs/)
- [un compte Github](https://github.com/join)
- [Node 18](https://nodejs.org/en/download) & [PHP 8.0](https://www.php.net/downloads.php)(optional)

### Installer Azure CLI

#### Si CLI non installée : Installer le CLI

Suivez [ce lien](https://docs.microsoft.com/fr-fr/cli/azure/install-azure-cli) et l'onglet correspondant à votre système d'exploitation.

#### Se connecter à sa souscription

{% collapsible %}

```bash
az login
# choisir la souscription 
az account set –s <SubscriptionID> 
# vérifier la souscription
az account show
```

connectez vous au portail : <https://portal.azure.com>

{% endcollapsible %}

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
- [Node 18](https://nodejs.org/en/download) et [PHP 8.0](https://www.php.net/downloads.php) (facultatif)

#### Installer Azure CLI

Suivez [ce lien](https://docs.microsoft.com/fr-fr/cli/azure/install-azure-cli) et l'onglet correspondant à votre système d'exploitation.

#### Se connecter à sa souscription avec Azure CLI et le [portail](<https://portal.azure.com>)

{% collapsible %}

```bash
az login
# choisir la souscription 
az account set –s <SubscriptionID> 
# vérifier la souscription
az account show
```

{% endcollapsible %}

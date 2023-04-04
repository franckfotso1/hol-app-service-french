---
sectionid: prereq
sectionclass: h2
title: Convention de nommage
parent-id: intro
---

Avant de commencer à déployer des ressources Azure, il est important de suivre une convention de nommage. Un nom adapté vous aide à identifier rapidement le type de la ressource, la charge de travail associée, l’environnement et la région Azure qui l’héberge. Sur la base de la [documentation][az-naming-convention] officielle, nous allons définir quelques composants:

- Le nom d'application: `hol` (Hands On Lab)
- Le type d'environnement: `dev`
- La région Azure: `we` (West Europe)
- Le nombre d'instances d'une ressource spécifique: `01`
- Le nom de l'organisation: `ms`

Nous utiliserons donc cette convention:

```xml
<!--If the resource prefix has a dash: -->
<service-prefix>-<environment>-<region>-<application-name>-<owner>-<instance>
<!--If the resource does not autorize any special caracters: -->
<service-prefix><environment><region><application-name><owner><instance>
```

> Assurez-vous d’utiliser vos propres valeurs pour avoir des noms uniques ou utilisez votre propre convention.<br>
> [Official resource abbreviations][az-abrevation]

Maintenant que tout est prêt, commençons le lab!

[az-cli-install]: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli
[az-func-core-tools]: https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Clinux%2Ccsharp%2Cportal%2Cbash#install-the-azure-functions-core-tools
[az-naming-convention]: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming
[az-abrevation]: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations
[az-portal]: https://portal.azure.com
[vs-code]: https://code.visualstudio.com/
[azure-function-vs-code-extension]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions
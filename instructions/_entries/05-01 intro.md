---
sectionid: lab4-intro
sectionclass: h2
title: Contexte
parent-id: lab-4
---

Par défaut, les applications hébergées dans App Service sont accessibles directement via Internet et peuvent atteindre uniquement des points de terminaison hébergés sur Internet. Toutefois, pour bon nombre d’applications, vous devez contrôler le trafic réseau entrant et sortant. App Service inclut plusieurs fonctionnalités destinées à vous aider à répondre à ces besoins.

La fonctionnalité VNet Integration (Intégration de réseau virtuel) d'Azure Web App permet de connecter votre application web à un réseau virtuel (VNet) Azure et d'accéder aux ressources de votre réseau virtuel. Cela peut être utile si vous voulez accéder à des ressources qui ne sont pas accessibles publiquement ou si vous voulez utiliser des connexions privées à vos services.
La fonctionnalité VNet integration sert uniquement à passer des appels sortants de votre application vers votre VNet. Utilisez un [Private Endpoint](https://learn.microsoft.com/en-us/azure/app-service/networking/private-endpoint) pour l’accès privé entrant

Dans ce Lab, vous allez configurer une communication sécurisée pour la Todo App.(voir schema Lab-2)

- Le trafic sortant à partir de App Service est acheminé vers le réseau virtuel (VNet) et peut atteindre les back-end services notamment Key Vault et CosmosDB.
- Le trafic public vers les back-end services est bloqué.
- Azure App Service utilise une identité managée pour se connecter au Key Vault ce qui permet d’éliminer les secrets de connexion à gérer et de sécuriser la connectivité back-end dans un environnement de production
- App Service effectue des appels à CosmosDB via des secrets reférencés dans le Key Vault.

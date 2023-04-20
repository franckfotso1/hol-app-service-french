---
sectionid: lab4-local cache
sectionclass: h2
title: Local cache
parent-id: lab-4
---

Certaines applications ont uniquement besoin d’un magasin de contenu en lecture seule très performant à partir duquel elles peuvent s’exécuter avec une haute disponibilité. Ces applications peuvent tirer profit d’une instance de machine virtuelle sur un cache local spécifique. Les applications qui s’exécutent sur le cache local bénéficient des avantages suivants :

- Elles sont protégées contre les latences qui se produisent quand elles accèdent au contenu sur Azure Storage.
- Elles ne redémarrent pas systématiquement après des modifications du partage de stockage.

#### Activez le cache local dans votre web App

> La taille des deux dossiers pour chaque application est limitée à 1 Go par défaut, mais vous pouvez l’augmenter à 2 Go

{% collapsible %}
![local cache](/media/lab3/local_cache.png)
{% endcollapsible %}

Pour en savoir plus sur comment le cache local change le comportement d'App service, consultez [ici](https://learn.microsoft.com/fr-fr/azure/app-service/overview-local-cache#how-the-local-cache-changes-the-behavior-of-app-service)

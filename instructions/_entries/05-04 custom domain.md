---
sectionid: lab4-custom domain
sectionclass: h2
title: Custom domain 
parent-id: lab-4
---


Le nom de domaine par défaut fourni avec votre application <app-name>.azurewebsites.net peut ne pas représenter votre marque comme vous le souhaitez. vous allez configurer App Service avec un domaine www dont vous êtes propriétaire, comme www.contoso.com, et sécuriser le domaine personnalisé avec un certificat managé App Service.

Le nom <app-name>.azurewebsites.net est déjà sécurisé par un certificat générique pour toutes les applications App Service, mais votre domaine personnalisé doit être sécurisé par TLS avec un certificat distinct.

#### Ajouter un nom de domaine personnalisé

{% collapsible %}
![custom domain](/media/lab3/custom_domain.png)
![go daddy](/media/lab3/godaddy.png)
{% endcollapsible %}

#### Ajouter un certificat

{% collapsible %}
![ssl certificate](/media/lab3/ssl_certificate.png)
![custom domain working](/media/lab3/custom_domain_working.png)
{% endcollapsible %}

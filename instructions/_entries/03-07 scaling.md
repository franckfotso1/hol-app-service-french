---
sectionid: lab2-scaling
sectionclass: h2
title: Autoscaling
parent-id: lab-2
---

Une fois le déploiement effectué, nous allons mettre à l'échelle l'application

Il existe deux types de mise à l’échelle dans App Service : le scale-up et le scale-out

{% collapsible %}
![Web App connection string](/media/lab2/asp_scaling.png)

> en faisant un **scale up**, vous bénéficiez d’un surcroît de capacité de vCPU et de mémoire. Pour monter en puissance il faut modifier le niveau tarifaire du plan App Service auquel appartient votre application.<br>
> en faisant un **scale out**, vous augmentez le nombre d’instances de machine virtuelle qui exécutent votre application. Vous y trouverez comment utiliser  la mise à l'echelle manuelle ou automatique basé sur des règles prédéfinies et en fonction des planifications.

{% endcollapsible %}

#### Mettez à jour le nombre de workers à 3 (scale out)

```bash
az appservice plan update --number-of-workers 3 --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP
```

#### Mettez à jour le tier de l'ASP pour rendre disponible la fonctionnalité de mise à l'échelle automatique basé sur des règles (scale up)

{% collapsible %}
![scale up](/media/lab2/scale_up_aspp.png)
{% endcollapsible %}

#### Configurez un 'Rule Based autoscale'
  
> La règle utilise le pourcentage de CPU pour augmenter le nombre d'instances. Si en moyenne, sur une durée de 5 min, cette métrique > 10%, on incrémente de 1 ce nombre en limitant à 5.

{% collapsible %}
![Add a scale Rule](/media/lab2/rule_based_autoscale.png)
{% endcollapsible %}

- Utilisez Azure CLI pour démarrer une boucle infinie qui envoie les requêtes HTTP à votre application Web.
  
```bash
$webapp= az webapp show --name $APP_NAME --resource-group $RESOURCE_GROUP --query defaultHostName -o tsv

while true
do
  curl https://$webapp
done
```

- Vérifiez bien que le nombre d'instances évolue automatiquement

Regardez la valeur du paramètre **Instance count** du plan App Service dans la vue globale

```bash
# s'il était à 3, il passera à 4, et ensuite 5
az appservice plan show -n $APP_SERVICE_PLAN  -g $RESOURCE_GROUP  --query 'sku.capacity'
```

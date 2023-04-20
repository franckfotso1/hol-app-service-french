---
sectionid: lab4-network
sectionclass: h2
title: Vnet integration 
parent-id: lab-4
---

A présent isolons le trafic pour que la Web App accède à la base de données via un réseau virtuel.

#### Créez un réseau virtuel (VNet)

{% collapsible %}

```bash
VNET_NAME="<virtual-network-name>"
az network vnet create -g $RESOURCE_GROUP -l "$LOCATION" --name $VNET_NAME --address-prefixes 10.0.0.0/16
```

{% endcollapsible %}

#### Créez un sous-réseau (Subnet) pour l’intégration VNet à App Service

{% collapsible %}

```bash
VNET_NAME="<virtual-network-name>"
az network vnet subnet create -g $RESOURCE_GROUP --vnet-name $VNET_NAME --name vnet-integration-subnet --address-prefixes 10.0.0.0/24 --delegations Microsoft.Web/serverfarms
```

{% endcollapsible %}

#### Activez l’intégration du VNet à votre Todo App

{% collapsible %}

```bash
az webapp vnet-integration add -g $RESOURCE_GROUP --name $APP_NAME --vnet $VNET_NAME --subnet vnet-integration-subnet
```

![web app vnet integration](/media/lab3/vnet_integration.png)
{% endcollapsible %}

> Cet intégration permet au trafic sortant de circuler directement dans le VNet

#### Configurez l'accès à votre base de données Cosmos DB depuis le Vnet

{% collapsible %}
![cosmos vnet integration](/media/lab3/vnet_integration.png)
{% endcollapsible %}

> Vous ne pouvez plus charger l’application, vous recevez une erreur HTTP 500. L’application a perdu sa connectivité à la ressource Cosmos DB. Pour rétablir la connexion en passant par le Vnet, il faut activer le service endpoint.

#### Ajoutez le service enpoint Microsoft.Azure.Cosmos DB à votre subnet

{% collapsible %}
![service endpoint](/media/lab3/service_endpoint.png)
{% endcollapsible %}

> Une fois le service endpoint activé, rafraichir la page et la connexion à la base de données est de nouveau rétablie.

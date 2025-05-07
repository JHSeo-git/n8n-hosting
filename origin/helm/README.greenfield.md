- https://learn.microsoft.com/ko-kr/azure/application-gateway/tutorial-ingress-controller-add-on-existing

```bash
AKS_NAME=aks-aoai-kc-axpg-dev-comm-01
RESOURCE_GROUP=rg-aoai-kc-aipg-dev
LOCATION='Korea Central'

APPGW_NAME="agw-aoai-kc-axpg-dev-01"
APPGW_SUBNET_NAME="sbn-aoai-kc-axpg-dev-agw-01"
APPGW_SUBNET_ID="/subscriptions/e2db5ede-f128-4a3e-87ce-43b3063b6d92/resourceGroups/rg-aoai-kc-aipg-dev/providers/Microsoft.
```

```bash
PUBLIC_IP_NAME="pip-aoai-kc-axpg-dev-01"
VNET_NAME="vnet-aoai-kc-axpg-dev-01"
SUBNET_NAME="sbn-aoai-kc-axpg-dev-01"

az network public-ip create --name $PUBLIC_IP_NAME --resource-group $RESOURCE_GROUP --allocation-method Static --sku Standard
az network vnet create --name $VNET_NAME --resource-group $RESOURCE_GROUP --address-prefix 10.0.0.0/16 --subnet-name $SUBNET_NAME --subnet-prefix 10.0.0.0/24
az network application-gateway create --name $APPGW_NAME --resource-group $RESOURCE_GROUP --sku Standard_v2 --public-ip-address $PUBLIC_IP_NAME --vnet-name $VNET_NAME --subnet $APPGW_SUBNET_ID --priority 100
```

```bash
appgwId=$(az network application-gateway show --name myApplicationGateway --resource-group myResourceGroup -o tsv --query "id")
az aks enable-addons --name myCluster --resource-group myResourceGroup --addon ingress-appgw --appgw-id $appgwId
```

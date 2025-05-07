## aks 생성

```bash
AKS_NAME='aks-axpg-n8n-dev-01'
RESOURCE_GROUP='rg-aoai-kc-aipg-dev'
LOCATION='Korea Central'
VM_SIZE='Standard_DS2_v2'

APPGW_NAME='agw-axpg-n8n-dev-01'

az aks create \
  --name $AKS_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --node-vm-size $VM_SIZE \
  --network-plugin azure \
  --enable-managed-identity \
  --enable-addons ingress-appgw \
  --appgw-name $APPGW_NAME \
  --appgw-subnet-cidr "10.225.0.0/16" \
  --generate-ssh-keys
```

## application gateway 정보

```bash
# Get application gateway id from AKS addon profile
appGatewayId=$(az aks show -n $AKS_NAME -g $RESOURCE_GROUP -o tsv --query "addonProfiles.ingressApplicationGateway.config.effectiveApplicationGatewayId")

# Get Application Gateway subnet id
appGatewaySubnetId=$(az network application-gateway show --ids $appGatewayId -o tsv --query "gatewayIPConfigurations[0].subnet.id")

# Get AGIC addon identity
agicAddonIdentity=$(az aks show -n $AKS_NAME -g $RESOURCE_GROUP -o tsv --query "addonProfiles.ingressApplicationGateway.identity.clientId")

# Assign network contributor role to AGIC addon identity to subnet that contains the Application Gateway
az role assignment create --assignee $agicAddonIdentity --scope $appGatewaySubnetId --role "Network Contributor"
```

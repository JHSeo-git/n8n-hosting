## helm

```bash
helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
```

### set workload identity

```bash
AKS_NAME=aks-aoai-kc-axpg-dev-comm-01
RESOURCE_GROUP=rg-aoai-kc-aipg-dev

az aks update -g rg-aoai-kc-aipg-dev -n aks-aoai-kc-axpg-dev-comm-01 --enable-oidc-issuer --enable-workload-identity --no-wait
```

### create application-gateway

```bash
AKS_NAME=aks-aoai-kc-axpg-dev-comm-01
RESOURCE_GROUP=rg-aoai-kc-aipg-dev
LOCATION='Korea Central'

APPGW_NAME="agw-aoai-kc-axpg-dev-01"
APPGW_SUBNET_NAME="sbn-aoai-kc-axpg-dev-agw-01"
APPGW_SUBNET_ID="/subscriptions/e2db5ede-f128-4a3e-87ce-43b3063b6d92/resourceGroups/rg-aoai-kc-aipg-dev/providers/Microsoft.Network/virtualNetworks/vnet-aoai-kc-axpg-dev-01/subnets/sbn-aoai-kc-axpg-dev-agw-01"

az network application-gateway create \
  --name "agw-aoai-kc-axpg-dev-01" \
  --location "Korea Central" \
  --resource-group rg-aoai-kc-aipg-dev \
  --subnet "/subscriptions/e2db5ede-f128-4a3e-87ce-43b3063b6d92/resourceGroups/rg-aoai-kc-aipg-dev/providers/Microsoft.Network/virtualNetworks/vnet-aoai-kc-axpg-dev-01/subnets/sbn-aoai-kc-axpg-dev-agw-01" \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address appgw-ip \
  --priority 10

APPGW_ID=$(az network application-gateway show --name "agw-aoai-kc-axpg-dev-01" --resource-group rg-aoai-kc-aipg-dev --query "id" --output tsv)
# /subscriptions/e2db5ede-f128-4a3e-87ce-43b3063b6d92/resourceGroups/rg-aoai-kc-aipg-dev/providers/Microsoft.Network/applicationGateways/agw-aoai-kc-axpg-dev-01
```

### create identity

```bash
AKS_NAME=aks-aoai-kc-axpg-dev-comm-01
RESOURCE_GROUP=rg-aoai-kc-aipg-dev
LOCATION='Korea Central'

IDENTITY_RESOURCE_NAME='id-aoai-kc-axpg-agic-dev-01'

echo "Creating identity $IDENTITY_RESOURCE_NAME in resource group $RESOURCE_GROUP"
az identity create --resource-group $RESOURCE_GROUP --name $IDENTITY_RESOURCE_NAME
IDENTITY_PRINCIPAL_ID="$(az identity show -g $RESOURCE_GROUP -n $IDENTITY_RESOURCE_NAME --query principalId -otsv)"
IDENTITY_CLIENT_ID="$(az identity show -g $RESOURCE_GROUP -n $IDENTITY_RESOURCE_NAME --query clientId -otsv)"

echo "Waiting 60 seconds to allow for replication of the identity..."
sleep 60

echo "Set up federation with AKS OIDC issuer"
AKS_OIDC_ISSUER="$(az aks show -n "$AKS_NAME" -g "$RESOURCE_GROUP" --query "oidcIssuerProfile.issuerUrl" -o tsv)"
az identity federated-credential create --name "agic" \
    --identity-name "$IDENTITY_RESOURCE_NAME" \
    --resource-group $RESOURCE_GROUP \
    --issuer "$AKS_OIDC_ISSUER" \
    --subject "system:serviceaccount:default:ingress-azure"

resourceGroupId=$(az group show --name $RESOURCE_GROUP --query id -otsv)
nodeResourceGroup=$(az aks show -n $AKS_NAME -g $RESOURCE_GROUP -o tsv --query "nodeResourceGroup")
nodeResourceGroupId=$(az group show --name $nodeResourceGroup --query id -otsv)

echo "Apply role assignments to AGIC identity"
az role assignment create --assignee-object-id $IDENTITY_PRINCIPAL_ID --assignee-principal-type ServicePrincipal --scope $resourceGroupId --role "Reader"
az role assignment create --assignee-object-id $IDENTITY_PRINCIPAL_ID --assignee-principal-type ServicePrincipal --scope $nodeResourceGroupId --role "Contributor"
az role assignment create --assignee-object-id $IDENTITY_PRINCIPAL_ID --assignee-principal-type ServicePrincipal --scope $APPGW_ID --role "Contributor"
```

```bash
helm install ingress-azure \
  oci://mcr.microsoft.com/azure-application-gateway/charts/ingress-azure \
  --set appgw.applicationGatewayID="/subscriptions/e2db5ede-f128-4a3e-87ce-43b3063b6d92/resourceGroups/rg-aoai-kc-aipg-dev/providers/Microsoft.Network/applicationGateways/agw-aoai-kc-axpg-dev-01" \
  --set armAuth.type=workloadIdentity \
  --set armAuth.identityClientID=e5ac8d39-292c-449c-a84f-1e4311217611 \
  --set rbac.enabled=true \
  --version 1.7.3

helm upgrade ingress-azure \
  oci://mcr.microsoft.com/azure-application-gateway/charts/ingress-azure \
  --set appgw.applicationGatewayID=$APPGW_ID \
  --set armAuth.type=workloadIdentity \
  --set armAuth.identityClientID=$IDENTITY_CLIENT_ID \
  --set rbac.enabled=true \
  --version 1.7.3
```

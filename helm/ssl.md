```bash
az ad sp create-for-rbac --role Contributor --scopes /subscriptions/e2db5ede-f128-4a3e-87ce-43b3063b6d92 -o json > auth.json
appId=$(jq -r ".appId" auth.json)
password=$(jq -r ".password" auth.json)
```

```bash
wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/deploy/azuredeploy.json -O template.json
```

```bash
resourceGroupName="rg-aoai-kc-aipg-dev"
location="Korea Central"
deploymentName="deployment-ingress-appgw"

# create a resource group
az group create -n $resourceGroupName -l $location

# modify the template as needed
az deployment group create -g $resourceGroupName -n $deploymentName --template-file template.json --parameters parameters.json
```

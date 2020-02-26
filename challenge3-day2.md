Below steps are based on https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration-cli



## 1/ create a subnet
```powershell
az network vnet subnet create -g teamResources --vnet-name vnet -n Hacker1AksSubnet --address-prefixes 10.2.2.0/24

$SUBNET_ID=$(az network vnet subnet show -g teamResources -n Hacker1AksSubnet --vnet-name vnet --query "id" --output tsv)
echo $SUBNET_ID
```

## 2/ Set the cluster name variable
```powershell
$clusterName="AKSClusterHacker1"
```

## 3/ Create serverApplicationId
```powershell
$serverApplicationId=$(az ad app create --display-name "${clusterName}Server" --identifier-uris "https://${clusterName}Server" --query appId -o tsv)
echo $serverApplicationId
```

## 4/ Crete serverApplicationSecret

```powershell
$serverApplicationSecret=$(az ad sp credential reset --name $serverApplicationId --credential-description "AKSPassword" --query password -o tsv)
echo $serverApplicationSecret
```

### 4.1 Assign these permissions using the az ad app permission add command:

```powershell
az ad app permission add --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000 --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope 06da0dbc-49e2-44d2-8312-53f166ab848a=Scope 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role
```

There will be a Yellow command output saying `Invoking .... is needed`, copy that command and execute. `This step fails if the current account is not a tenant admin`

### 4.2 Grant the permissions assigned in the previous step for the server application
```powershell    
az ad app permission grant --id $serverApplicationId --api 00000003-0000-0000-c000-000000000000
az ad app permission admin-consent --id  $serverApplicationId
```

### 4.3 Optional, use below command to see all permissions
```powershell 
az ad app permission list --id $serverApplicationId
```

## 5/ Create Azure AD client component
```powershell 
$clientApplicationId=$(az ad app create --display-name "${clusterName}Client" --native-app --reply-urls "https://${clusterName}Client" --query appId -o tsv)
echo $clientApplicationId
```

## 6/ Create a service principal for the client application using the az ad sp create command:
```powershell 
az ad sp create --id $clientApplicationId
```

## 7/ Get the oAuth2 ID for the server app to allow the authentication flow between the two app components using the az ad app show command. This oAuth2 ID is used in the next step.
```powershell 
$oAuthPermissionId=$(az ad app show --id $serverApplicationId --query "oauth2Permissions[0].id" -o tsv)
echo $oAuthPermissionId
```


## 8/ Add the permissions for the client application and server application components to use the oAuth2 communication flow using the az ad app permission add command. Then, grant permissions for the client application to communication with the server application using the az ad app permission grant command:
```powershell 
az ad app permission add --id $clientApplicationId --api $serverApplicationId --api-permissions ${oAuthPermissionId}=Scope
#Execute the yellow command in the output or bellow command
az ad app permission grant --id $clientApplicationId --api $serverApplicationId
```

## 9/ Deploy the cluster
```powershell
$tenantId=$(az account show --query tenantId -o tsv)
echo $tenantId

#NOTE: the service-cidr must not conflict with the created subnet range at step1
#NOTE: https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal
#Create Service Pricipal manually

az ad sp create-for-rbac --skip-assignment --name "${clusterName}ServicePrincipal" --query appId -o tsv
##Grab the appId and password to use in the next command

az aks create --resource-group teamResources --name $clusterName --generate-ssh-keys --network-plugin azure --vnet-subnet-id $SUBNET_ID --docker-bridge-address 172.17.0.1/16 --service-cidr 10.3.0.0/16 --dns-service-ip 10.3.0.10 --aad-server-app-id $serverApplicationId --aad-server-app-secret $serverApplicationSecret --aad-client-app-id $clientApplicationId --aad-tenant-id $tenantId --service-principal 5cfc1996-376a-4a2b-8c77-99063c6a70da --client-secret 13b391a2-5747-401e-a39a-748034136680 --network-policy azure
```



## 10/ https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration-cli#create-rbac-binding

```powershell
az aks get-credentials --resource-group teamResources --name $clusterName --admin
#Get the user principal name (UPN) for the user currently logged in using the az ad signed-in-user show command.
az ad signed-in-user show --query userPrincipalName -o tsv
```

Create a YAML manifest named basic-azure-ad-binding.yaml and paste the following contents. On the last line, replace userPrincipalName_or_objectId with the UPN or object ID output from the previous command:

There are 2 different RBAC, make yourself an admin using this binding is different to the Admin in IAM in Azure portal
This yaml allow you to do everything INSIDE the cluster like pods, etc not the Azure Portal

Example: allow developer to create name space, pods but can't delete the Cluster resource

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: hacker1-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: hacker1l9o@OTAPRD209ops.onmicrosoft.com
```

then execute the binding

```powershell
kubectl apply -f basic-azure-ad-binding.yaml
```


## 11/ Get the admin crential once the cluster is created successfully

```powershell
az aks get-credentials --resource-group teamResources --name $clusterName --overwrite-existing --admin
kubectl get pods --all-namespaces
kubectl exec <pod_name> -- ping <private_ip>
```
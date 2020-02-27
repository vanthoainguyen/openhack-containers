Following steps are based on https://docs.microsoft.com/en-us/azure/aks/ingress-basic

## 1. Check heml
```powershell
helm version
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm search repo stable
helm repo update
```

## 2. Add Service acc and role binding for Tiller service

https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm#install-an-application-with-helm-v2

## 3. Configure helm (doesn't work ???)
```powershell
helm init --history-max 200 --service-account tiller --node-selectors "beta.kubernetes.io/os=linux"
```

## 4. Create an ingress controller withx Azure Load balancer

https://docs.microsoft.com/en-us/azure/aks/ingress-basic

```powershell
kubectl create namespace ingress-basic

helm install nginx-ingress stable/nginx-ingress --namespace ingress-basic --set controller.replicaCount=2 --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

kubectl get service -l app=nginx-ingress --namespace ingress-basic
```



## 5. Generate chart for poi api (not required by the challenge, TODO)
https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/
```powershell
cd \src\poi
helm create helm-chart

#Open values.yaml, update
# version to 1.0.0
# 
```


## 6. Install all apis
Once all the apis installed, get the internal ip of those pods
```powershell
kubectl apply -f all-apis.yaml --namespace api

kubectl get pods --namespace api
kubectl get pod trips-78bbdc5dfc-g9kw6 -o wide --namespace api
kubectl get pod userprofile-bddb685d5-b42tp -o wide --namespace api

#Then update those ips to the tripviewer.yaml env variables

kubectl apply -f tripviewer.yaml --namespace web
```

## 7.Create an ingress route


```powershell
kubectl apply -f tripviewer-ingress.yaml
kubectl apply -f api-ingress.yaml

#if you need delete and reapply
kubectl delete Ingress web-ingress --namespace web
kubectl get Ingress --all-namespaces

```

public ingress ip: 20.43.106.255


## Other notes:

- SecretRef is the K8s thing, nothing todo with Azure (check Challenge 3)
- kubectl descirbe pod poi-59d8b75ddb-rcd6t --namespace api
- Go to resource group to see a magic resource group created by AKS to manage nodes, scale set
- Keyvault flex .... use option 4 is recommended (user dont see the secret), option1 is created first. Dont ever use option pod itentity
- Check jpda.dev blog



kubectl delete deployment userprofile --namespace api
kubectl delete deployment user-java --namespace api 
kubectl delete deployment trips --namespace api
kubectl delete deployment poi --namespace api

kubectl delete service userprofile --namespace api
kubectl delete service user-java --namespace api
kubectl delete service trips --namespace api
kubectl delete service poi --namespace api 

## Control access to cluster resources using RBAC

###### group names
web-dev
api-dev
 

AKS_ID=$(az aks show \
    --resource-group teamResources \
    --name newakscluster \
    --query id -o tsv)
 
## AKS ID
(base) Renes-MacBook-Pro:openhack-containers franckess$ echo $AKS_ID
/subscriptions/27db2cbd-64d3-41bd-96f3-770b03afd259/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/newakscluster
 

## APPDEV_ID
APPDEV_ID=$(az ad group create --display-name web-dev --mail-nickname web-dev --query objectId -o tsv)
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ APPDEV_ID=$(az ad group create --display-name web-dev --mail-nickname web-dev --query objectId -o tsv)
(base) Renes-MacBook-Pro:openhack-containers franckess$ echo $APPDEV_ID
f878dce4-35ef-41fd-8867-e7863284fb15
(base) Renes-MacBook-Pro:openhack-containers franckess$
 
######
az role assignment create \
  --assignee $APPDEV_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ az role assignment create \
>   --assignee $APPDEV_ID \
>   --role "Azure Kubernetes Service Cluster User Role" \
>   --scope $AKS_ID
{
  "canDelegate": null,
  "id": "/subscriptions/27db2cbd-64d3-41bd-96f3-770b03afd259/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/newakscluster/providers/Microsoft.Authorization/roleAssignments/9911df2b-51c3-4225-88b1-dd756a82c112",
  "name": "9911df2b-51c3-4225-88b1-dd756a82c112",
  "principalId": "f878dce4-35ef-41fd-8867-e7863284fb15",
  "principalType": "Group",
  "resourceGroup": "teamResources",
  "roleDefinitionId": "/subscriptions/27db2cbd-64d3-41bd-96f3-770b03afd259/providers/Microsoft.Authorization/roleDefinitions/4abbcc35-e782-43d8-92c5-2d3f1bd2253f",
  "scope": "/subscriptions/27db2cbd-64d3-41bd-96f3-770b03afd259/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/newakscluster",
  "type": "Microsoft.Authorization/roleAssignments"
}
(base) Renes-MacBook-Pro:openhack-containers franckess$ 
 

#####
OPSSRE_ID=$(az ad group create --display-name api-dev --mail-nickname api-dev --query objectId -o tsv)
 
base) Renes-MacBook-Pro:openhack-containers franckess$ OPSSRE_ID=$(az ad group create --display-name api-dev --mail-nickname api-dev --query objectId -o tsv)
(base) Renes-MacBook-Pro:openhack-containers franckess$ echo $OPSSRE_ID
fa7e703a-46bc-4aaa-a9ec-5dacd80c350d
(base) Renes-MacBook-Pro:openhack-containers franckess$ 
 

######
az role assignment create \
  --assignee $OPSSRE_ID \
  --role "Azure Kubernetes Service Cluster User Role" \
  --scope $AKS_ID
 
#### Hacker 1 member id
a7a6a35e-4192-4716-b1b7-6586581eabff
az ad group member add --group appdev --member-id $AKSDEV_ID
az ad group member add --group web-dev --member-id a7a6a35e-4192-4716-b1b7-6586581eabff
 
#### Hacker 2 member id
68846e00-74c2-4518-820d-6f666d1dcf95
az ad group member add --group web-dev --member-id 68846e00-74c2-4518-820d-6f666d1dcf95
 
#### Hacker 3 member id
087c4e92-aa0f-4fb9-a34e-a6d52d446676
az ad group member add --group api-dev --member-id 087c4e92-aa0f-4fb9-a34e-a6d52d446676
 
#### Hacker 4 member id
1577b9ac-6888-4295-a3ae-85a0c0e399b0
az ad group member add --group api-dev --member-id 1577b9ac-6888-4295-a3ae-85a0c0e399b0
 
#### Hacker 5 member id
2c84f9fa-52e3-41f8-9da3-bc20f7a93bb8
 

##### Admin step
az aks get-credentials --resource-group teamResources --name newakscluster --admin
 

####### Create a role deployment
kubectl apply -f api_role.yml
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ kubectl apply -f api_role.yml
role.rbac.authorization.k8s.io/api-user-full-access created
(base) Renes-MacBook-Pro:openhack-containers franckess$ 
 

##### Get object-id/resource_id of the group web-dev
az ad group show --group web-dev --query objectId -o tsv
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ az ad group show --group web-dev --query objectId -o tsv
f878dce4-35ef-41fd-8867-e7863284fb15
(base) Renes-MacBook-Pro:openhack-containers franckess$
 
##### YAML file role binding
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: api-user-access
  namespace: api
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: api-user-full-access
subjects:
- kind: Group
  namespace: api
  name: f878dce4-35ef-41fd-8867-e7863284fb15
 
##### 
kubectl apply -f ApiRoleBinding.yml 
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ kubectl apply -f ApiRoleBinding.yml
rolebinding.rbac.authorization.k8s.io/api-user-access created
(base) Renes-MacBook-Pro:openhack-containers franckess$
 

#######
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: web-user-full-access
  namespace: web
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]
 

kubectl apply -f web_role.yml
 
Rolebinding.rbac.authorization.k8s.io/api-user-access created
(base) Renes-MacBook-Pro:openhack-containers franckess$ kubectl apply -f web_role.yml
role.rbac.authorization.k8s.io/web-user-full-access created
(base) Renes-MacBook-Pro:openhack-containers franckess$ 
 

####### Object ID of web
az ad group show --group web-dev --query objectId -o tsv
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ az ad group show --group web-dev --query objectId -o tsv
f878dce4-35ef-41fd-8867-e7863284fb15
(base) Renes-MacBook-Pro:openhack-containers franckess$ 
 
######
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
name: api-dev-rolebinding
namespace: web
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: ClusterRole
name: view
subjects:
- kind: Group
namespace: web
name: f878dce4-35ef-41fd-8867-e7863284fb15
apiGroup: rbac.authorization.k8s.io
 
kubectl apply -f WebRoleBinding.yml
 
(base) Renes-MacBook-Pro:openhack-containers franckess$ kubectl apply -f WebRoleBinding.yml
rolebinding.rbac.authorization.k8s.io/web-user-access created
(base) Renes-MacBook-Pro:openhack-containers franckess$
 
Group Role Binding

 
#### API (yaml file)
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

 name: api-dev-rolebinding

 namespace: api

roleRef:

 apiGroup: rbac.authorization.k8s.io

 kind: ClusterRole

 name: Edit

subjects:

- kind: Group

 namespace: api

 name: f878dce4-35ef-41fd-8867-e7863284fb15

---

 kind: RoleBinding

 apiVersion: rbac.authorization.k8s.io/v1

 metadata:

 name: api-dev-rolebinding

 namespace: web

 roleRef:

 apiGroup: rbac.authorization.k8s.io

 kind: ClusterRole

 name: view

 subjects:

 - kind: Group

 namespace: web

 name: f878dce4-35ef-41fd-8867-e7863284fb15

 apiGroup: rbac.authorization.k8s.io
 
#### WEB (yaml file)
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

 name: web-dev-rolebinding

 namespace: web

roleRef:

 apiGroup: rbac.authorization.k8s.io

 kind: ClusterRole

 name: Edit

subjects:

- kind: Group

 namespace: web

 name: f878dce4-35ef-41fd-8867-e7863284fb15

---

 kind: RoleBinding

 apiVersion: rbac.authorization.k8s.io/v1

 metadata:

 name: web-dev-rolebinding

 namespace: api

 roleRef:

 apiGroup: rbac.authorization.k8s.io

 kind: ClusterRole

 name: view

 subjects:

 - kind: Group

 namespace: api

 name: f878dce4-35ef-41fd-8867-e7863284fb15

 apiGroup: rbac.authorization.k8s.io


 ## Group Role Binding Yaml

 ```yaml
 apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

 name: api-dev-rolebinding

 namespace: api

roleRef:

 apiGroup: rbac.authorization.k8s.io

 kind: ClusterRole

 name: Edit

subjects:

- kind: Group

 namespace: api

 name: f878dce4-35ef-41fd-8867-e7863284fb15
 ```


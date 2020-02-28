
## Create AKS cluster using az cmd
```powershell
az login
az aks create --resource-group teamResources --name openHackTeam11AKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
az aks install-cli
```

## configure azure credential for kubectl
```powershell
az aks get-credentials --resource-group teamResources --name openHackTeam11AKSCluster
kubectl get nodes
```
## Configure ACR integration for existing AKS clusters
```powershell
az aks update -n openHackTeam11AKSCluster -g teamResources --attach-acr registryrnr1073
```
## Deploy all apis and web

The challenge2-day2.yaml is the manifest contains everything that need to be deployed.
Each api/web has a deployment yaml and a service yaml.
You can separeted those or simply put everything in 1 and kubectl apply

```powershell
kubectl apply -f challenge2-day2.yaml


#watch creation
kubectl get service --watch

#useful cmd
kubectl delete deployment trips
kubectl delete svc trips
kubectl get pods
kubectl delete pods poi-id
```

## Each service has a public load balancer with public ip so you can test from browser

```
http://20.193.20.8/api/poi
http://20.40.187.54/api/trips
http://20.40.124.34/api/user
http://20.40.185.157/api/user-java/healthcheck
http://20.40.191.79/trip
```
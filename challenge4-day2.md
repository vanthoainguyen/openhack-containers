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
- Keyvault flex .... use option 4 is recommended (user dont see the secret), option1 is created first 

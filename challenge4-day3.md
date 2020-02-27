Day-3
az login
az aks get-credentials --resource-group teamResources --name newakscluster --admin
 
\\Get cluster admin credentials 
PS C:\Users\nspr2> az aks get-credentials --resource-group teamResources --name newakscluster --admin
Merged "newakscluster-admin" as current context in C:\Users\nspr2\.kube\config
 
\\Deploy Key Vault FlexVolume to your AKS cluster with this command:
PS C:\Users\nspr2> kubectl create -f https://raw.githubusercontent.com/Azure/kubernetes-keyvault-flexvol/master/deployment/kv-flexvol-installer.yaml
namespace/kv created
daemonset.apps/keyvault-flexvolume created
 
PS C:\Users\nspr2> kubectl get pods -n kv
NAME                        READY   STATUS    RESTARTS   AGE
keyvault-flexvolume-24hbs   1/1     Running   0          3m12s
keyvault-flexvolume-jtcfd   1/1     Running   0          3m12s
keyvault-flexvolume-v67wk   1/1     Running   0          3m12s
PS C:\Users\nspr2> 
 
az aks create \
    --resource-group teamResources \
    --name $aksname \
    --generate-ssh-keys \
    --network-plugin azure \
    --vnet-subnet-id "/subscriptions/27db2cbd-64d3-41bd-96f3-770b03afd259/resourceGroups/teamResources/providers/Microsoft.Network/virtualNetworks/vnet/subnets/MySubnet" \
    --docker-bridge-address 172.17.0.1/16 \
    --service-cidr 10.3.0.0/16 \
    --dns-service-ip 10.3.0.10 \
    --aad-server-app-id 855bccc6-b78e-4180-ba49-b5e6abd3caae \
    --aad-server-app-secret 7efd48c9-5e33-4e82-9470-91bc568f65bc \
    --aad-client-app-id 1e765d19-69f5-4d20-9006-b600fa946a8b \
    --aad-tenant-id e5ff0039-b551-41b2-90e1-e230cc49eeae \
    --service-principal ad42291c-70d9-4fda-a22c-d2019b29638e \
    --client-secret 0a26f33e-b493-42d2-af7e-5ee2c42ad67f \
    --network-policy azure
    
 

****************Create a key vault************
https://docs.microsoft.com/en-us/azure/key-vault/quick-create-cli
 
az keyvault create --name "TripKeyVault" --resource-group "teamResources" --location eastus
PS C:\Karthik\OpenHack\Karthik> az keyvault create --name "KeyVault-Team11" --resource-group "teamResources" --location eastus
{
  "id": "/subscriptions/27db2cbd-64d3-41bd-96f3-770b03afd259/resourceGroups/teamResources/providers/Microsoft.KeyVault/vaults/KeyVault-Team11",
  "location": "eastus",
  "name": "KeyVault-Team11",
  "properties": {
    "accessPolicies": [
      {
        "applicationId": null,
        "objectId": "087c4e92-aa0f-4fb9-a34e-a6d52d446676",
        "permissions": {
          "certificates": [
            "get",
            "list",
            "delete",
            "create",
            "import",
            "update",
            "managecontacts",
            "getissuers",
            "listissuers",
            "setissuers",
            "deleteissuers",
            "manageissuers",
            "recover"
          ],
          "keys": [
            "get",
            "create",
            "delete",
            "list",
            "update",
            "import",
            "backup",
            "restore",
            "recover"
          ],
          "secrets": [
            "get",
            "list",
            "set",
            "delete",
            "backup",
            "restore",
            "recover"
          ],
          "storage": [
            "get",
            "list",
            "delete",
            "set",
            "update",
            "regeneratekey",
            "setsas",
            "listsas",
            "getsas",
            "deletesas"
          ]
        },
        "tenantId": "e5ff0039-b551-41b2-90e1-e230cc49eeae"
      }
    ],
    "createMode": null,
    "enablePurgeProtection": null,
    "enableSoftDelete": null,
    "enabledForDeployment": false,
    "enabledForDiskEncryption": null,
    "enabledForTemplateDeployment": null,
    "networkAcls": null,
    "privateEndpointConnections": null,
    "provisioningState": "Succeeded",
    "sku": {
      "name": "standard"
    },
    "tenantId": "e5ff0039-b551-41b2-90e1-e230cc49eeae",
    "vaultUri": "https://keyvault-team11.vault.azure.net/"
  },
  "resourceGroup": "teamResources",
  "tags": {},
  "type": "Microsoft.KeyVault/vaults"
}
PS C:\Karthik\OpenHack\Karthik> 
 
***************Create secrets inside the keyvault******************
        - name: SQL_SERVER
          value: "sqlserverrnr1073.database.windows.net"
        - name: SQL_DBNAME
          value: "mydrivingDB"
        - name: SQL_PASSWORD
          value: "xR3dc3Em3"
        - name: SQL_USER
          value: "sqladminrNr1073" 
 
az keyvault secret set --vault-name "keyvault-team11" --name "SQLSERVER" --value "sqlserverrnr1073.database.windows.net"
az keyvault secret set --vault-name "keyvault-team11" --name "SQLDBNAME" --value "mydrivingDB"
az keyvault secret set --vault-name "keyvault-team11" --name "SQLPASSWORD" --value "xR3dc3Em3"
az keyvault secret set --vault-name "keyvault-team11" --name "SQLUSER" --value "sqladminrNr1073"
 
***************************************************************************
 
**********************************************************************************************************
 
az vmss identity show -g <resource group>  -n <vmss scalset name> -o yaml
 
az vmss identity show -g MC_teamResources_newakscluster_australiaeast -n aks-nodepool1-84572546-vmss -o yaml
 
az vmss identity assign -g MC_teamResources_newakscluster_australiaeast -n aks-nodepool1-84572546-vmss 
PS C:\Karthik\OpenHack\Karthik> az vmss identity assign -g MC_teamResources_newakscluster_australiaeast -n aks-nodepool1-84572546-vmss
With manual upgrade mode, you will need to run 'az vmss update-instances -g MC_teamResources_newakscluster_australiaeast -n aks-nodepool1-84572546-vmss --instance-ids *' to propagate the change
{
  "systemAssignedIdentity": "0639e858-a1e1-4612-9374-55607fee7c64",
  "userAssignedIdentities": {}
}
PS C:\Karthik\OpenHack\Karthik> 
PS C:\Karthik\OpenHack\Karthik> az vmss identity show -g MC_teamResources_newakscluster_australiaeast -n aks-nodepool1-84572546-vmss
{
  "principalId": "0639e858-a1e1-4612-9374-55607fee7c64",
  "tenantId": "e5ff0039-b551-41b2-90e1-e230cc49eeae",
  "type": "SystemAssigned",
  "userAssignedIdentities": null
}
PS C:\Karthik\OpenHack\Karthik> 
 
# set policy to access keys in your Key Vault
az keyvault set-policy -n KeyVault-Team11 --key-permissions get --object-id 0639e858-a1e1-4612-9374-55607fee7c64
# set policy to access secrets in your Key Vault
az keyvault set-policy -n KeyVault-Team11 --secret-permissions get --object-id 0639e858-a1e1-4612-9374-55607fee7c64
# set policy to access certs in your Key Vault
az keyvault set-policy -n KeyVault-Team11 --certificate-permissions get --object-id 0639e858-a1e1-4612-9374-55607fee7c64
 
*****************************************************************************************************************
 
### Use this yaml to deploy apis again with Keyvault
```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: poi
  namespace: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: poi
  template:
    metadata:
      labels:
        app: poi
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: poi
        image: registryrnr1073.azurecr.io/tripinsights/poi:1.0
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        volumeMounts:
        - name: keyvault
          mountPath: /secrets
          readOnly: true
      volumes:
      - name: keyvault
        flexVolume:
          driver: "azure/kv"
          options:
            keyvaultname: "KeyVault-Team11"
            usevmmanagedidentity: "true"            
            keyvaultobjectnames: "SQLSERVER;SQLDBNAME;SQLPASSWORD;SQLUSER"
            keyvaultobjecttypes: "secret;secret;secret;secret"
            keyvaultobjectaliases: "SQL_SERVER;SQL_DBNAME;SQL_PASSWORD;SQL_USER"
            resourcegroup: "teamResources"
            subscriptionid: "27db2cbd-64d3-41bd-96f3-770b03afd259"
            tenantid: "e5ff0039-b551-41b2-90e1-e230cc49eeae"      
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trips
  namespace: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: trips
  template:
    metadata:
      labels:
        app: trips
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: trips
        image: registryrnr1073.azurecr.io/tripinsights/trips:1.0
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        volumeMounts:
        - name: keyvault
          mountPath: /secrets
          readOnly: true  
      volumes:
      - name: keyvault
        flexVolume:
          driver: "azure/kv"
          options:
            keyvaultname: "KeyVault-Team11"
            usevmmanagedidentity: "true"            
            keyvaultobjectnames: "SQLSERVER;SQLDBNAME;SQLPASSWORD;SQLUSER"
            keyvaultobjecttypes: "secret;secret;secret;secret"
            keyvaultobjectaliases: "SQL_SERVER;SQL_DBNAME;SQL_PASSWORD;SQL_USER"
            resourcegroup: "teamResources"
            subscriptionid: "27db2cbd-64d3-41bd-96f3-770b03afd259"
            tenantid: "e5ff0039-b551-41b2-90e1-e230cc49eeae"      
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: userprofile
  namespace: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: userprofile
  template:
    metadata:
      labels:
        app: userprofile
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: userprofile
        image: registryrnr1073.azurecr.io/tripinsights/userprofile:1.0
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        volumeMounts:
        - name: keyvault
          mountPath: /secrets
          readOnly: true     
      volumes:
      - name: keyvault
        flexVolume:
          driver: "azure/kv"
          options:
            keyvaultname: "KeyVault-Team11"
            usevmmanagedidentity: "true"            
            keyvaultobjectnames: "SQLSERVER;SQLDBNAME;SQLPASSWORD;SQLUSER"
            keyvaultobjecttypes: "secret;secret;secret;secret"
            keyvaultobjectaliases: "SQL_SERVER;SQL_DBNAME;SQL_PASSWORD;SQL_USER"
            resourcegroup: "teamResources"
            subscriptionid: "27db2cbd-64d3-41bd-96f3-770b03afd259"
            tenantid: "e5ff0039-b551-41b2-90e1-e230cc49eeae"       
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-java
  namespace: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-java
  template:
    metadata:
      labels:
        app: user-java
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: user-java
        image: registryrnr1073.azurecr.io/tripinsights/user-java:1.0
        resources:
          requests:
            cpu: 500m
            memory: 256Mi
          limits:
            cpu: 1000m
            memory: 512Mi
        ports:
        - containerPort: 80
        volumeMounts:
        - name: keyvault
          mountPath: /secrets
          readOnly: true   
      volumes:
      - name: keyvault
        flexVolume:
          driver: "azure/kv"
          options:
            keyvaultname: "KeyVault-Team11"
            usevmmanagedidentity: "true"            
            keyvaultobjectnames: "SQLSERVER;SQLDBNAME;SQLPASSWORD;SQLUSER"
            keyvaultobjecttypes: "secret;secret;secret;secret"
            keyvaultobjectaliases: "SQL_SERVER;SQL_DBNAME;SQL_PASSWORD;SQL_USER"
            resourcegroup: "teamResources"
            subscriptionid: "27db2cbd-64d3-41bd-96f3-770b03afd259"
            tenantid: "e5ff0039-b551-41b2-90e1-e230cc49eeae"        
---
apiVersion: v1
kind: Service
metadata:
  name: poi
  namespace: api
spec:
  ports:
  - port: 80
  selector:
    app: poi
---
apiVersion: v1
kind: Service
metadata:
  name: trips
  namespace: api
spec:
  ports:
  - port: 80
  selector:
    app: trips
---
apiVersion: v1
kind: Service
metadata:
  name: userprofile
  namespace: api
spec:
  ports:
  - port: 80
  selector:
    app: userprofile
---
apiVersion: v1
kind: Service
metadata:
  name: user-java
  namespace: api
spec:
  ports:
  - port: 80
  selector:
    app: user-java            
```
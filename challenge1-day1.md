## Create bridge network so all container can communicate

```powershell
docker network create my_network
```

All containers attached to this network can communicate to eachother using container name

## Run local MSSQL
```powershell
docker run --network my_network -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=Passw0rd' -p 1433:1433 --name changeme.database.windows.net -d mcr.microsoft.com/mssql/server:2017-latest
#note that changeme.database.windows.net is actually the docker contaniner name, not the dns or anything
#note the default sa password from the command is Passw0rd
```

## create db
```powershell
docker ps
#grab the container id
docker exec -it <sql-contaner-id> "bash"

#then run
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Passw0rd'

CREATE DATABASE mydrivingDB
GO

# exit the bash
```


## Load the SQL seed data
```powershell
docker run --network my_network -e SQLFQDN="changeme.database.windows.net" -e SQLUSER="sa" -e SQLPASS="Passw0rd" -e SQLDB=mydrivingDB openhack/data-load:v1
#note that the container openhack/data-load:v1 is attached to the same network and communicate with server changeme.database.windows.net which is the name of mssql container instance
```


## Build containers

For each project, in each folder run bellow steps & commands

```powershell
#update each dockerfile and set the sql credential
docker build -t poi-hacker1 .
docker build -t trips-hacker1 .
docker build -t user-java-hacker1 .
docker build -t userprofile-hacker1 .
#note, you will need to update the docker file for tripviewer and set trips & userprofile as the default endpoints in the ENV
docker build -t tripviewer-hacker1 .
```




## Run the apps

```powershell
docker run -d --network my_network -e "ASPNETCORE_ENVIRONMENT=Local" -p 8083:80 --name poi poi-hacker1
curl http://localhost:8083/api/poi/healthcheck

docker run -d --network my_network -e "ASPNETCORE_ENVIRONMENT=Local" -p 8082:80 --name userprofile userprofile-hacker1
curl http://localhost:8082/api/user/healthcheck


docker run -d --network my_network -e "ASPNETCORE_ENVIRONMENT=Local" -p 8084:80 --name trips trips-hacker1
curl http://localhost:8084/api/trips/healthcheck


docker run -d --network my_network -e "ASPNETCORE_ENVIRONMENT=Local" -p 8079:80 --name user-java user-java-hacker1
curl http://localhost:8079/api/user-java/healthcheck


docker run -d --network my_network -e "ASPNETCORE_ENVIRONMENT=Local" -p 8081:80 --name tripviewer tripviewer-hacker1
curl http://localhost:8081
```




## Tags all images
```powershell
# assume you go to azure and create the openhackteam11 ACR
docker tag userprofile-hacker1 openhackteam11.azurecr.io/userprofile-hacker1:v1
docker tag poi-hacker1 openhackteam11.azurecr.io/poi-hacker1:v1
docker tag trips-hacker1 openhackteam11.azurecr.io/trips-hacker1:v1
docker tag tripviewer-hacker1 openhackteam11.azurecr.io/tripviewer-hacker1:v1
docker tag user-java-hacker1 openhackteam11.azurecr.io/user-java-hacker1:v1
```


## Push imagse to ACR

```powershell
az acr login --name openhackteam11


docker push openhackteam11.azurecr.io/userprofile-hacker1:v1
docker push openhackteam11.azurecr.io/poi-hacker1:v1
docker push openhackteam11.azurecr.io/trips-hacker1:v1
docker push openhackteam11.azurecr.io/tripviewer-hacker1:v1
docker push openhackteam11.azurecr.io/user-java-hacker1:v1

```

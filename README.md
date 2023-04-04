## Creer Dockerfile

``` bash
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build-env

WORKDIR /App


COPY  DotNet.Docker.csproj .

RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o out


FROM mcr.microsoft.com/dotnet/aspnet:7.0

WORKDIR /App

COPY --from=build-env /App/out .

ENTRYPOINT [ "dotnet", "DotNet.Docker.dll" ]

```
## Push image 

``` bash
Docker build -t azankoudm/app
docker push azankoudm/app

```

## create file deployment 
 ``` bash
 apiVersion: apps/v1
# On spécifie la version de l'API que l'on va utiliser pour notre fichier de déploiement 
 
kind: Deployment
# On spécifie quel type de ressource on est en train de créer
 
metadata:
  name: mon-app
# On spécifie via les méta-données le nom qu'aura notre déploiement dans le cluster
 
spec: # On défini ensuite les spécificités du / des pods
  replicas: 5 # On peut définir directement le nombre de répliques de notre pod que l'on veut
  selector:
    matchLabels:
      app: mon-app
  template: 
    metadata:
      labels: # Dans les labels, on peut définir autant d'ensemble clé: valeur que l'on veut
        app: mon-app
    spec:
      containers: # On défini ensuite les conteneurs de notre pod
        - name: mon-app # On donne le nom du conteneur 
          image: azankoudm/app:latest # ON spécifie l'image du conteneur
          resources: # On spécifie les ressources utilisées par le conteneur
            limits: # On fixe les limites
              memory: "128Mi" # De RAM
              cpu: "500m" # De temps d'utilisation du processeur
          ports: # On défini les ports utilisés par le conteneur
            - containerPort: 80
```

## Service

 ```bash
 apiVersion: v1 # La version de l'API pour le contexte du fichier
kind: Service # Le type de ressource que l'on défini
metadata: # Les méta-données de la ressource
  name: mon-app
spec:
  selector: # ON sélectionne par label (ceux ci doivent correspondre avec ceux des pods)
    app: mon-app
  type: LoadBalancer # On défini le type de service
  ports: # On défini le port forwarding (port -> targetPort)
  - port: 80
    targetPort: 5000
 ```

kubectl apply -f .\ressources-files\service.yml,.\ressources-files\deployment.yml

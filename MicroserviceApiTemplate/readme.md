# Guide d'utilisation du Dockerfile et du Template .NET

## Introduction

Ce projet fournit un modèle de projet .NET Web API préconfiguré pour l'exécution dans un conteneur Docker avec le débogage intégré. Il inclut :

- Un `Dockerfile` pour construire et exécuter l'application .NET dans un conteneur.
- Un fichier `template.json` permettant de créer un modèle de projet .NET personnalisé.
- Une configuration optimisée pour l'intégration avec Visual Studio pour un débogage efficace.

---

## Dockerfile : Construction et Exécution

### Étapes de Build

1. **Construire l'image Docker**

   ```sh
   docker build -t app:v1.0 -f ./MicroserviceApiTemplate/Dockerfile .
   ```

2. **Exécuter le conteneur en mode HTTP**

   ```sh
   docker run --rm -it -p 8000:8081 -e ASPNETCORE_HTTP_PORTS=8081 app:v1.0
   ```

3. **Configurer HTTPS** (optionnel)

   ```sh
   dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p password
   dotnet dev-certs https --trust
   ```

4. **Exécuter en mode HTTPS**

   ```sh
   docker run --rm -it -p 8000:8081 \
      -e ASPNETCORE_HTTPS_PORTS=8081 \
      -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" \
      -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx \
      -v ${HOME}/.aspnet/https:/https/ app:v1.0
   ```

5. **Activer Swagger en mode développement**

   ```sh
   -e ASPNETCORE_ENVIRONMENT=Development
   ```

### Débogage avec Visual Studio

Visual Studio utilise `Dockerfile.debug` pour générer les images de conteneur, accélérant ainsi le débogage.

**Lien utile** : [Personnalisation des conteneurs pour le débogage](https://aka.ms/customizecontainer)

---

## Utilisation de l'image Docker depuis Docker Hub

L'image Docker est disponible sur [Docker Hub](https://hub.docker.com/repository/docker/zaelyndra/microserviceapitemplate/general). Vous pouvez l'utiliser directement avec :

```sh
docker run --rm -it -p 8000:8081 -e ASPNETCORE_ENVIRONMENT=Development \
    -e ASPNETCORE_HTTP_PORTS=8081 zaelyndra/microserviceapitemplate:latest
```

Pour exécuter une version spécifique, remplacez `latest` par le tag correspondant :

```sh
docker run --rm -it -p 8000:8081 -e ASPNETCORE_ENVIRONMENT=Development \
    -e ASPNETCORE_HTTP_PORTS=8081 zaelyndra/microserviceapitemplate:{tag}
```

**Liste des tags disponibles** : [Docker Hub du projet](https://hub.docker.com/repository/docker/zaelyndra/microserviceapitemplate/general)

---

## Utilisation du Template .NET

### Installation du modèle

À la racine du projet, exécutez :

- **Windows**
  ```sh
  dotnet new install .\
  ```
- **Linux/macOS**
  ```sh
  dotnet new install ./
  ```

### Désinstallation du modèle

- **Windows**
  ```sh
  dotnet new uninstall .\
  ```
- **Linux/macOS**
  ```sh
  dotnet new uninstall ./
  ```

### Création d'un projet à partir du modèle

- **Afficher l'aide sur le modèle**
  ```sh
  dotnet new microsapi -?
  ```
- **Générer un projet avec un nom personnalisé**
  ```sh
  dotnet new microsapi -n {PROJECT_NAME}
  ```

---

## Liens Utiles

**Docker et .NET**

- [Construction d'images avec Visual Studio](https://learn.microsoft.com/fr-fr/visualstudio/containers/container-build?view=vs-2022)
- [Exemples d'images Docker .NET](https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md)
- [Sécurisation HTTPS dans les conteneurs ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-9.0)

**Templates .NET**

- [Créer un modèle d'élément .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tutorials/cli-templates-create-item-template)
- [Référence du fichier template.json](https://github.com/dotnet/templating/wiki/Reference-for-template.json)
- [Contraintes des modèles .NET](https://github.com/dotnet/templating/wiki/Constraints)
- [Personnalisation des modèles .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tools/custom-templates)





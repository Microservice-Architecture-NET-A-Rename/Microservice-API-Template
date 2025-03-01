# Guide d'utilisation du Dockerfile et du Template .NET

## Introduction

Ce projet fournit un mod�le de projet .NET Web API pr�configur� pour l'ex�cution dans un conteneur Docker avec le d�bogage int�gr�. Il inclut :

- Un `Dockerfile` pour construire et ex�cuter l'application .NET dans un conteneur.
- Un fichier `template.json` permettant de cr�er un mod�le de projet .NET personnalis�.
- Une configuration optimis�e pour l'int�gration avec Visual Studio pour un d�bogage efficace.

---

## Dockerfile : Construction et Ex�cution

### �tapes de Build

1. **Construire l'image Docker**

   ```sh
   docker build -t app:v1.0 -f ./MicroserviceApiTemplate/Dockerfile .
   ```

2. **Ex�cuter le conteneur en mode HTTP**

   ```sh
   docker run --rm -it -p 8000:8081 -e ASPNETCORE_HTTP_PORTS=8081 app:v1.0
   ```

3. **Configurer HTTPS** (optionnel)

   ```sh
   dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p password
   dotnet dev-certs https --trust
   ```

4. **Ex�cuter en mode HTTPS**

   ```sh
   docker run --rm -it -p 8000:8081 \
      -e ASPNETCORE_HTTPS_PORTS=8081 \
      -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" \
      -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx \
      -v ${HOME}/.aspnet/https:/https/ app:v1.0
   ```

5. **Activer Swagger en mode d�veloppement**

   ```sh
   -e ASPNETCORE_ENVIRONMENT=Development
   ```

### D�bogage avec Visual Studio

Visual Studio utilise `Dockerfile.debug` pour g�n�rer les images de conteneur, acc�l�rant ainsi le d�bogage.

**Lien utile** : [Personnalisation des conteneurs pour le d�bogage](https://aka.ms/customizecontainer)

---

## Utilisation de l'image Docker depuis Docker Hub

L'image Docker est disponible sur [Docker Hub](https://hub.docker.com/repository/docker/zaelyndra/microserviceapitemplate/general). Vous pouvez l'utiliser directement avec :

```sh
docker run --rm -it -p 8000:8081 -e ASPNETCORE_ENVIRONMENT=Development \
    -e ASPNETCORE_HTTP_PORTS=8081 zaelyndra/microserviceapitemplate:latest
```

Pour ex�cuter une version sp�cifique, remplacez `latest` par le tag correspondant :

```sh
docker run --rm -it -p 8000:8081 -e ASPNETCORE_ENVIRONMENT=Development \
    -e ASPNETCORE_HTTP_PORTS=8081 zaelyndra/microserviceapitemplate:{tag}
```

**Liste des tags disponibles** : [Docker Hub du projet](https://hub.docker.com/repository/docker/zaelyndra/microserviceapitemplate/general)

---

## Utilisation du Template .NET

### Installation du mod�le

� la racine du projet, ex�cutez :

- **Windows**
  ```sh
  dotnet new install .\
  ```
- **Linux/macOS**
  ```sh
  dotnet new install ./
  ```

### D�sinstallation du mod�le

- **Windows**
  ```sh
  dotnet new uninstall .\
  ```
- **Linux/macOS**
  ```sh
  dotnet new uninstall ./
  ```

### Cr�ation d'un projet � partir du mod�le

- **Afficher l'aide sur le mod�le**
  ```sh
  dotnet new microsapi -?
  ```
- **G�n�rer un projet avec un nom personnalis�**
  ```sh
  dotnet new microsapi -n {PROJECT_NAME}
  ```

---

## Liens Utiles

**Docker et .NET**

- [Construction d'images avec Visual Studio](https://learn.microsoft.com/fr-fr/visualstudio/containers/container-build?view=vs-2022)
- [Exemples d'images Docker .NET](https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md)
- [S�curisation HTTPS dans les conteneurs ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-9.0)

**Templates .NET**

- [Cr�er un mod�le d'�l�ment .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tutorials/cli-templates-create-item-template)
- [R�f�rence du fichier template.json](https://github.com/dotnet/templating/wiki/Reference-for-template.json)
- [Contraintes des mod�les .NET](https://github.com/dotnet/templating/wiki/Constraints)
- [Personnalisation des mod�les .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tools/custom-templates)





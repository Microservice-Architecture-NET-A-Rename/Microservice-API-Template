# Guide d'utilisation du Dockerfile et du Template .NET pour une application .NET

## Introduction
Ce projet contient :
- Un `Dockerfile` permettant de construire et exécuter une application .NET dans un conteneur.
- Un fichier `template.json` permettant de créer un modèle de projet .NET personnalisé.


## Dockerfile : Construire et Exécuter l'Image

### Étapes de Build

1. **Construire l'image Docker**
   ```sh
   docker build -t app:v1.0 .
   ```
2. **Exécuter le conteneur en mode HTTP**
   ```sh
   docker run --rm -it -p 8000:8081 -e ASPNETCORE_HTTP_PORTS=8081 app:v1.0
   ```
3. **Configurer HTTPS** (facultatif)
   ```sh
   dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p password
   dotnet dev-certs https --trust
   ```
   - Placez le répertoire `.aspnet` dans votre projet.

4. **Exécuter en mode HTTPS**
   ```sh
   docker run --rm -it -p 8000:8081 \  
      -e ASPNETCORE_HTTPS_PORTS=8081 \  
      -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" \  
      -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx \  
      -v ${HOME}\.aspnet\https:/https/ app:v1.0
   ```
   
5. **Activer Swagger en mode développement**
   ```sh
   -e ASPNETCORE_ENVIRONMENT=Development
   ```

### Débogage avec Visual Studio

Visual Studio utilise le `Dockerfile.debug` pour générer les images de conteneur, ce qui accélère le débogage. 
Pour plus d'informations sur la personnalisation du conteneur de débogage, consultez la documentation officielle :
[Personnalisation des conteneurs pour le débogage](https://aka.ms/customizecontainer).

### Liens Utiles

- [Construction d'images de conteneurs avec Visual Studio](https://learn.microsoft.com/fr-fr/visualstudio/containers/container-build?view=vs-2022)
- [Exemples d'images Docker .NET](https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md)
- [Sécurisation HTTPS dans les conteneurs ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-9.0)


## Utilisation du Template .NET

### Installation du modèle

À la racine du projet, exécutez la commande appropriée selon votre OS :

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

- Afficher l'aide sur le modèle :
  ```sh
  dotnet new microsapi -?
  ```
- Générer un nouveau projet avec un nom personnalisé :
  ```sh
  dotnet new microsapi -n {PROJECT_NAME}
  ```

### Liens Utiles
- [Créer un modèle d'élément .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tutorials/cli-templates-create-item-template)
- [Référence du fichier template.json](https://github.com/dotnet/templating/wiki/Reference-for-template.json)
- [Contraintes des modèles .NET](https://github.com/dotnet/templating/wiki/Constraints)
- [Personnalisation des modèles .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tools/custom-templates)




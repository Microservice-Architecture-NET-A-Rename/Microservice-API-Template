# Guide d'utilisation du Dockerfile et du Template .NET pour une application .NET

## Introduction
Ce projet contient :
- Un `Dockerfile` permettant de construire et ex�cuter une application .NET dans un conteneur.
- Un fichier `template.json` permettant de cr�er un mod�le de projet .NET personnalis�.


## Dockerfile : Construire et Ex�cuter l'Image

### �tapes de Build

1. **Construire l'image Docker**
   ```sh
   docker build -t app:v1.0 .
   ```
2. **Ex�cuter le conteneur en mode HTTP**
   ```sh
   docker run --rm -it -p 8000:8081 -e ASPNETCORE_HTTP_PORTS=8081 app:v1.0
   ```
3. **Configurer HTTPS** (facultatif)
   ```sh
   dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p password
   dotnet dev-certs https --trust
   ```
   - Placez le r�pertoire `.aspnet` dans votre projet.

4. **Ex�cuter en mode HTTPS**
   ```sh
   docker run --rm -it -p 8000:8081 \  
      -e ASPNETCORE_HTTPS_PORTS=8081 \  
      -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" \  
      -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx \  
      -v ${HOME}\.aspnet\https:/https/ app:v1.0
   ```
   
5. **Activer Swagger en mode d�veloppement**
   ```sh
   -e ASPNETCORE_ENVIRONMENT=Development
   ```

### D�bogage avec Visual Studio

Visual Studio utilise le `Dockerfile.debug` pour g�n�rer les images de conteneur, ce qui acc�l�re le d�bogage. 
Pour plus d'informations sur la personnalisation du conteneur de d�bogage, consultez la documentation officielle :
[Personnalisation des conteneurs pour le d�bogage](https://aka.ms/customizecontainer).

### Liens Utiles

- [Construction d'images de conteneurs avec Visual Studio](https://learn.microsoft.com/fr-fr/visualstudio/containers/container-build?view=vs-2022)
- [Exemples d'images Docker .NET](https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md)
- [S�curisation HTTPS dans les conteneurs ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-9.0)


## Utilisation du Template .NET

### Installation du mod�le

� la racine du projet, ex�cutez la commande appropri�e selon votre OS :

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

- Afficher l'aide sur le mod�le :
  ```sh
  dotnet new microsapi -?
  ```
- G�n�rer un nouveau projet avec un nom personnalis� :
  ```sh
  dotnet new microsapi -n {PROJECT_NAME}
  ```

### Liens Utiles
- [Cr�er un mod�le d'�l�ment .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tutorials/cli-templates-create-item-template)
- [R�f�rence du fichier template.json](https://github.com/dotnet/templating/wiki/Reference-for-template.json)
- [Contraintes des mod�les .NET](https://github.com/dotnet/templating/wiki/Constraints)
- [Personnalisation des mod�les .NET CLI](https://learn.microsoft.com/fr-fr/dotnet/core/tools/custom-templates)




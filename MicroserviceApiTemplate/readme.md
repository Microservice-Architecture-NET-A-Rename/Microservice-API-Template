# Guide d'utilisation du Dockerfile pour une application .NET

## Introduction
Ce projet contient un `Dockerfile` permettant de construire et ex�cuter une application .NET dans un conteneur. 

Le fichier est con�u pour :
- Construire une image .NET 8.0 sur Alpine Linux.
- S�parer les �tapes de build et d'ex�cution pour optimiser la taille de l'image.
- G�rer les modes HTTP et HTTPS correctement.

## Construire et Ex�cuter l'Image

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
   docker run --rm -it -p 8000:80 -p 8001:443 \  
      -e ASPNETCORE_URLS="https://+;http://+" \  
      -e ASPNETCORE_HTTPS_PORTS=8001 \  
      -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" \  
      -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx \  
      -v ${HOME}/.aspnet/https:/https/ app:v1.0
   ```

5. **Activer Swagger en mode d�veloppement**
   ```sh
   -e ASPNETCORE_ENVIRONMENT=Development
   ```

## D�bogage avec Visual Studio

Visual Studio utilise ce `Dockerfile` pour g�n�rer les images de conteneur, ce qui acc�l�re le d�bogage. 
Pour plus d'informations sur la personnalisation du conteneur de d�bogage, consultez la documentation officielle :
[Personnalisation des conteneurs pour le d�bogage](https://aka.ms/customizecontainer).

## Liens Utiles

- [Construction d'images de conteneurs avec Visual Studio](https://learn.microsoft.com/fr-fr/visualstudio/containers/container-build?view=vs-2022)
- [Exemples d'images Docker .NET](https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md)
- [S�curisation HTTPS dans les conteneurs ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-9.0)

## Licence
Ce projet est sous licence MIT. Voir le fichier `LICENSE` pour plus de d�tails.

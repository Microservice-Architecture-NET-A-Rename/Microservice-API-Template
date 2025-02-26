# Learn about building .NET container images:
# https://github.com/dotnet/dotnet-docker/blob/main/samples/README.md
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0-alpine AS build
ARG TARGETARCH
WORKDIR /source

# Copy project file and restore as distinct layers
COPY --link *.csproj .
RUN ls -l /source # Add this line for debugging
RUN dotnet restore -a $TARGETARCH

# Copy source code and publish app
COPY --link . .
RUN dotnet publish -a $TARGETARCH -o /app


# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine 
WORKDIR /app
COPY --link --from=build /app .
USER $APP_UID
ENTRYPOINT ["./MicroserviceApiTemplate"]


# IMAGE DE PRODUCTION: 
# COMMANDE 
# docker run --rm -it -p 8000:8081 -e ASPNETCORE_HTTP_PORTS=8081 app:v1.0
# dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p password   
#                  dotnet dev-certs https --trust
# Dans votre Home directory copié .aspnet et placer le repertoire dans le projet
# Build: docker build -t app:v1.0 .
# Start :
#   HTTP Mode : "docker run --rm -it -p 8000:8081 -e ASPNETCORE_HTTP_PORTS=8081  app:v1.0"
#   HTTPS Mode: "docker run --rm -it -p 8001:443 -e ASPNETCORE_URLS="https://+" -e ASPNETCORE_HTTPS_PORTS=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}\.aspnet\https:/https/ app:v1.0"
# Comme on veut afficher le swagger, il faut definir que l'on est en environnement de dev donc il faut rajouter: -e ASPNETCORE_ENVIRONMENT=Development
# 
#
###############################################################################################


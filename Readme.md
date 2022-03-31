# Creando una aplicación C# MultiPlataforma
Este ejemplo es una aplicación desarrollada en c# que permite hacer todo el ciclo de vida de la compilación y la ejecución usando contenedores docker. 

Esto provee de una gran ventaja operativa ya que no requiere que tener instalado en los entornos de despliegue las herramientas de compilación y empaquetado. 

## ¿Dónde lo podemos usar?  

Podemos aplicar este sistema a casi a cualquier plantilla moderna generada desde el aplicativo dotnet o para desarrollo realizados desde un IDE siempre y cuando se pueda usar la base de .Net Core. 

## Creación del App 

Para la creación de este articulo nos hemos basado en un ejemplo base que se genera desde el CLI dotnet. 

 
 
``` bash 
# nueva app usando la plantilla de webapp 
# proyecto nombre PizzeryMama 
# No usar https 
# usa el framework base net 5.0 
> dotnet new webapp -o PizzeryMama --no-https -f net5.0

```
## Compilación y empaquetado 

El proceso de compilación y empaquetado se hace todo desde dentro de imágenes docker lo que dota de independencia del entorno base (Windows, Linux, Mac) desde el que se ejecute siempre que este sea capad de ejecutar docker. 

``` Dockerfile

# https://hub.docker.com/_/microsoft-dotnet
FROM mcr.microsoft.com/dotnet/sdk:5.0-alpine AS build
WORKDIR /source

# copy csproj and restore as distinct layers

COPY *.csproj .
RUN dotnet restore -r linux-musl-x64 /p:PublishReadyToRun=true

# copy everything else and build app
COPY . .
RUN dotnet publish -c release -o /app -r linux-musl-x64 --self-contained true --no-restore /p:PublishTrimmed=true /p:PublishReadyToRun=true /p:PublishSingleFile=true

# final stage/image
FROM mcr.microsoft.com/dotnet/runtime-deps:5.0-alpine-amd64
WORKDIR /app
COPY --from=build /app ./
ENTRYPOINT ["./PizzeryMama"]

```
### Este proceso le dividimos en dos fases.  

En la primera Fase pasamos el proyecto dentro del contenedor. Para el empaquetado de la herramienta se usa la plataforma Docker para ello se ha genera un contenedor con las dependencias de compilación y SDK. 

En la segunda fase empaquetamos la base en un contenedor que contiene el compilado y Runtime base de ejecución .Net Core. 

 

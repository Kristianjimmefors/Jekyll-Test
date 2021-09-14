---
layout: post
title: Containers och docker
subtitle: Uppgift 3
cover-img: /assets/img/cloud-show.PNG
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Uppgifter, Internet, automation]
---

#Docker med github actions

Jag har installerat [docker desktop](https://docs.docker.com/get-docker/) på datorn

Applikationen kör i en container genom att läsa igenom en fil som heter Dockerfile för att sätta upp och köra en container

## Docker filen

Docker filen är ifrån ett av mina github repon som heter [Dockertest](https://github.com/Kristianjimmefors/Dockertest)

Här börjar vi med att initiera en ny base image som vi kallar för base och sedan sätts Working directory till app. Efter det så sätts vilka portar som docker lyssnar på när applikationen körs.

``` 
FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

Nu initierar vi en ny base image som vi kallar för build och sätter Working directory till src. Efter det är gjort så kopieras .csproj filen ifrån src mappen till WebApplication1 mappen i containern och så körs dotnet restore på .csproj filen för att återställa dependencies och verktyg för projektet. Nu kopieras alla filer ifrån samma mapp där dockerfilen ligger och kopierar det till ```/src/WebApplication1``` i containern. När allt detta har gjorts så körs dotnet build för att bygga applikationen.
```
FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["WebApplication1/WebApplication1.csproj", "WebApplication1/"]
RUN dotnet restore "WebApplication1/WebApplication1.csproj"
COPY . .
WORKDIR "/src/WebApplication1"
RUN dotnet build "WebApplication1.csproj" -c Release -o /app/build
```


När applikationen har byggts färdigt så initieras en ny base image ifrån build som kördes före, sedan körs dotnet publish vilket publicerar applikationen och alla dependencies till en mapp för distribution. 
```
FROM build AS publish
RUN dotnet publish "WebApplication1.csproj" -c Release -o /app/publish
```

Till sist så kör vi samma base image som kördes först och så sätts working directory till app i containern. Efter det så kopieras allt ifrån publish mappen till ```/app/publish``` i containern och sist så konfigureras entrypoints för containern som är körbar
```
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]
```


## Github pipeline

Det finns en workflow fil som heter [dotnet.yml](https://github.com/Kristianjimmefors/Dockertest/blob/main/.github/workflows/dotnet.yml) om man vill kolla på den, det den gör är att den bygger .NET applikationen.

DockerTest workflow filen aktiveras när man pushar till main branchen.
```
name: DockerTest

on:
  push:
    branches: main
```

Nu checkas koden ut ifrån github till workflown kan koden kan användas. 
```
jobs:
  login:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2.3.4
```

Efter det så loggar man in på github container registry så att man kan pusha sin docker image dit.
```
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.CR_PAT }}
```

Här pushar man en docker image till github container registry som inehåller applikationen.
```
    - name: Build and push Docker images
      uses: docker/build-push-action@v2.7.0
      with:
        push: true
        context: ${{env.working-directory}}
        tags: |
          ghcr.io/kristianjimmefors/dockertest:latest
          ghcr.io/kristianjimmefors/dockertest:${{ github.run_number }}
```

Det är hit som docker imagen pushas och även dit man går för att hämta hem den senaste docker imagen.
```
          ghcr.io/kristianjimmefors/dockertest:latest
          ghcr.io/kristianjimmefors/dockertest:${{ github.run_number }}
```

## Github action secrets

För att kunna logga in på Hithub Container Registry måste man skapa en personal access token. Det gör man genom att gå in på konto inställningar och sedan finns det under developer settings. Skriv ner eller kopiera strängen med text som har genererats för den försvinner när sidan har uppdaterats.

Här har jag skapat en personal access token som heter DockerTest
![Github ACT](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/github-act.PNG)

Efter man har skapat en personal access token (och kopierat den) så går man in på inställningar i sitt github repo och sedan secrets. Här skapar man sina action secrets. Som bilden visar så har jag skapt en action secret som heter CR_PAT och den kan man uppdatera med en ny personal access token när den har gått ut

![Github Action Secrets](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/github-action-secret.PNG)


---
layout: post
title: Containers och docker
subtitle: Uppgift 3
cover-img: /assets/img/CI-Actions.PNG
thumbnail-img: /assets/img/Devops.jpg
share-img: /assets/img/Devops.jpg
categories: [Uppgifter, Internet, automation]
---

Jag har installerat [docker desktop](https://docs.docker.com/get-docker/) på datorn

Applikationen kör i en container genom att läsa igenom en fil som heter Dockerfile för att sätta upp och köra en container

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

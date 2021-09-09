---
layout: post
title: Automatisering av bygg och release
subtitle: Uppgift 2
cover-img: /assets/img/CI-Actions.PNG
thumbnail-img: /assets/img/Devops.jpg
share-img: /assets/img/Devops.jpg
categories: [Uppgifter, Internet, automation]
---

## Vad är en CI pipline
CI är en förkortning för Continuous integration. Continuous integration är en kodsningsfilosofi och en uppsättning metoder för att driva utvecklingen framåt med små förändringar i koden och pusha det till sin kodbas. Målet med Continuous integration är att etablera ett konsekvent och automatiserat sätt att bygga, paketera och testa applikationer. Fördelen med Continuous integration är att man inte behöver sitta och hålla på med stora merge conflicts när man har byggt färdigt en funktion som kan ha tagit flera veckor att bygga, istället så pushar man ofta till sin kodbas så ser man snabbt om det har blivit något fel eller problem i koden och gör det lättare att fixa felen som har uppstått.

## Github actions workflow
Jag har gett workflowen namnent build .NET så man ser snabbt vad denna workflowen gör och den körs både när man pushar och gör en pull request till main branchen. Den kör på en linux VM på den senaste versionen och sedan kör den alla stegen som behövs för att starta en .Net verition 5.0.x vilket blir den senaste veritionen och sedan återställer alla dependencies som finns i projektet och sedan bygger den projektet. Den ska även köra testerna som finns i projektet men då måste den använda docker för köra databasen och jag har inte fått det att funka så jag har bara kommenterat ut den delen. Projektet har inte en mock databas för testerna vilket man borde använda.

`- uses: actions/checkout@v2` gör så checkar ut repot så att workflowen kan komma åt det.

```yml
- name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 5.0.x
```
Detta sätter upp en .NET miljö som kör den senaste versionen av 5.0

```yml
    - name: Restore dependencies
      working-directory: Source
      run: dotnet restore
    - name: Build
      working-directory: Source
      run: dotnet build --no-restore
  #- name: Test
  #  working-directory: Source/SpacePortTest
  #  run: dotnet test --no-build --verbosity normal
```
Här återställs alla dependencies som finns i projektet som ligger i Source mappen och sedan bygger den projektet. Testerna är för tillfället utkommenterade för att den failar alltid för vissa tester använder sig utav en databas som ligger i docker och jag har inte fått det att funka att köra docker i mitt workflow för tillfället.


### Workflow fil
````yml
name: Build .NET

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 5.0.x
    - name: Restore dependencies
      working-directory: Source
      run: dotnet restore
    - name: Build
      working-directory: Source
      run: dotnet build --no-restore
    #- name: Test
    #  working-directory: Source/SpacePortTest
    #  run: dotnet test --no-build --verbosity normal
````

Referenser som jag har använt:

[What is Continuous Integration?](https://www.youtube.com/watch?v=1er2cjUq1UI)

[About continuous integration](https://docs.github.com/en/actions/guides/about-continuous-integration)

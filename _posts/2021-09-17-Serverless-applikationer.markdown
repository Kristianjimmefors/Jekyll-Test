---
layout: post
title: Internet och Moln
subtitle: Uppgift 1
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, serverless]
---

## Serverless

Serverless eller serverless Computing som det även kallas, är en datormodell där infrastrukturen hanteras av en service provider som tilldelar maskin resurser på begäran av kunden. Detta betyder att man själv inte behöver hantera en fysisk hårdvara och kan skala upp och ner eller ta bort helt om man har bebov utav det. När appen inte används finns det inga datorresurser tilldelade appen. Detta göra att de datorresurser som en app inte använder kan användas av någon annan app. När appen kör så används resuserna för att göra alla beräkningar som behövs och sedan sparar den resultatet av körningarna. Prissättningen baseras på den faktiska mängden resurser som applikationen använder.

## Function As A Service (FaaS)

Function As A Service eller FaaS som det förkortas till är ett koncept som är till för att erbjuda utvecklare friheten att enkelt kunna utveckla, köra och hantera applikationsfunktioner i en molnmiljö. Detta gör det enklare för kunder att hantera sina applikationer i molnmiljön och slipper komplexiteten med att bygga och underhålla infraskrukturen som vanligtvis förknippas med utveckling och lansering av appar. Genom att bygga en applikation på detta sättet är ett sätt för att upnå en "serverless" arkitektur. FaaS används vanligtvis när man bygger microservices.

## Kalkylator koden

Koden är ganska simple den behöver två siffror som input och kollar kollar om dem kan omvandlas till Int's eller inte. Om det inte funkar att konvertera inputten till siffror så svarar den dem bad request och annars så svarar den med 200 ok. ```AuthorizationLevel.Admin``` är till för att skydda applikationen så att bara någon med en admin key kan använda appen.
```C#
public static class Calculator
    {
        [FunctionName("Calculator")]
        public static async Task<IActionResult> Run([HttpTrigger(AuthorizationLevel.Admin, "get", "post", Route = null)] HttpRequest req, ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string num1 = req.Query["num1"];
            string num2 = req.Query["num2"];

            string requestBody = String.Empty;
            using (StreamReader streamReader = new StreamReader(req.Body))
            {
                requestBody = await streamReader.ReadToEndAsync();
            }
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            num1 ??= data?.num1;
            num2 ??= data?.num2;

            if (int.TryParse(num1, out _) && int.TryParse(num2, out _))
            {

                int result = int.Parse(num1 ?? data?.num1) + int.Parse(num2 ?? data?.num2);

                return new OkObjectResult("Result: " + result);
            }

            return new BadRequestObjectResult("Please send two values");
        }
    }
```

## Pusha till Azure.

Jag tog och skapade upp ett Visual studio projekt med en mall för Azure function app. När man har byggt en Azure funktioin app i Visual studio så kan man pusha den till Azure via Build fliken högsupp i programmet. När man gjort det är det bara logga in på sitt Azure konto och följa instruktionerna. Efter det så kan man logga in på Azures hemsida för att se den. Efter det så kan det se ut så här i Visual studio.

[Image to push to Azure](https://github.com/Kristianjimmefors/Programmerings-grottan/blob/main/assets/img/Asure-push.PNG)

Så här ser det ut på Azures sida.

[Azure sidan](https://github.com/Kristianjimmefors/Programmerings-grottan/blob/main/assets/img/Function-app.PNG)

## Testing

Jag testade köra appen på localhost med Postman och fick det att funka men när jag testade på min Azure sida och då fick jag bara svar med en HTML sida som säger att Azure appen funkar.

[Localhost testing](https://github.com/Kristianjimmefors/Programmerings-grottan/blob/main/assets/img/Localhost-testing.PNG)

[Azure testing](https://github.com/Kristianjimmefors/Programmerings-grottan/blob/main/assets/img/Azure-test.PNG)

## Säkerhet

Jag har använt mig av Azure keys och satt så bara personer med rätt admin nyckel får använda API:et. Vilket funkar bra så länge man bara ska använda API:et privat eller av ett få antal personer. Om andra ska använda API:et så måste dom få Admin nyckeln vilket man helst inte ska dela ut på grund av säkerhetsrisker.

### Referenser

[Serverless computing](https://en.wikipedia.org/wiki/Serverless_computing)

[Function as a service](https://en.wikipedia.org/wiki/Function_as_a_service)

[HTTP Trigger Azure Function App](https://www.c-sharpcorner.com/article/how-to-create-an-http-trigger-azure-function-app-using-visual-studio-20172/)

[Postman](https://www.postman.com/)

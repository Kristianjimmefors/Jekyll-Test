---
layout: post
title: Filer i målnet
subtitle: Uppgift 8
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, Molnet]
---

## Monitorering av molnapplikationer

Appen som jag har laggt till monitorering på är den som gjordes i uppgift 6. Det är en todo list applikation som visar ens todos och så kan man lägga till och ta bort todos. Här är en bild som visar hur applikationen är kopplad till databasen och med applicatin insight.

![Application connecntions]()

## koden
Om ni vill kolla på koden för hela applikationen och inte bara för loggingen så läs [Webappar i målnet](https://kristianjimmefors.github.io/Programmerings-grottan/internet/m%C3%A5ln/2021/09/24/Webb-appar-molnet.html) där applikationen är beskriven.

installera Microsoft.ApplicationiInsights.AspNetCore nuget paketet i applikationen.
```C#
 <ItemGroup>
     <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.17.0" />
 </ItemGroup>
```

Lägg sedan till ```services.AddApplicationInsightsTelemetry();``` i startup.cs. Detta gör så att application insights blir anslutet atomatiskt när programmet startar.
```C#
public void ConfigureServices(IServiceCollection services)
        {
            services.AddApplicationInsightsTelemetry();
        }
```

I ```appsettings.json``` ska man lägga till detta så att man slipper skicka in sin instument nyckell ifrån application insight i ```AddApplicationInsightsTelemetry``` anroppet.
```C#
"ApplicationInsights": {
        "InstrumentationKey": "putinstrumentationkeyhere"
```

Här är ett väldigt simplet exemple på hur man kan använda loggning. Här används ```_logger.LogWarning``` för att logga warningar som kan förekomma när man kör applikationen och ```_logger.LogError``` för att logga errors som kan förekomma.
```C#
public class ValuesController : ControllerBase
{
    private readonly ILogger _logger;

    public ValuesController(ILogger<ValuesController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public ActionResult<IEnumerable<string>> Get()
    {
        _logger.LogWarning("An example of a Warning trace..");
        _logger.LogError("An example of an Error level message");

        return new string[] { "value1", "value2" };
    }
}
```

## Säkerhet med loggning

Man kan logga alla ändringar till databasen som med vem som gör dem och när dem är gjorda och vart ifrån dem är gjorda. Detta gör att om någon tagit bort något som inte dem inte ska ha tillgång till så kan man se vem det var, när det gjordes och vart ifrån. Det gör det möjligt för en att kunna lättare leta upp varför någon som inte ska ha behörighet kan göra detta. Loggning gör det möjligt för utvecklare att kunna undersöka errors som uppstår i applikationen lättare eftersom att man har ett medelande som man kan gå tillbaks till för att veta vad för error det var. Detta gör att man kan lättare fixa applikationen så att den funkar bättre och så att ingen ska kunna utnyttja vissa errors i applikationen.

## Kusto queries

Den första queryn som jag har använt visar tiden som det tar från att användaren har gjort en request till sidan tills DOMen, stylesheets, scripts och bilder är laddade. Den visar även en timechart så man får ett fin diagram. Datan man får fram här är intressant eftersom att om det tar mer än 3 sekunder att ladda sidan så är det mycket fler personer som inte kommer använda den.
```
browserTimings
| where isnotempty(totalDuration)
| extend _sum = totalDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

Den andra queryn som använts är till för att visa misslyckade operationer. Den beräkna hur många gånger operationer misslyckades och hur många användare som påverkades. Det är väldigt bra att se hur många misslyckade operationer som finns och hur många användare det påverkar så man kan lösa problemet och hjälpa så många användare som möjligt.
```
requests
| where success == false
| summarize failedCount=sum(itemCount), impactedUsers=dcount(user_Id) by operation_Name
| order by failedCount desc
```

## Referenser

[Application Insights for ASP.NET Core applications](https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core)

[Application Insights log-based metrics](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/app-insights-metrics)

---
layout: post
title: Skalning up och ut
subtitle: Uppgift 10
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, Skalning]
---

## Skala horisontalt och vertikalt

![Vertikal och horisontell skalning](https://www.webairy.com/wp-content/uploads/2019/07/hvsv.jpg)
[Bild ifrån webairy.com](https://www.webairy.com/wp-content/uploads/2019/07/hvsv.jpg)

### Skala Vertikalt

Skala vertikalt eller skala upp som det även heter så ökar man resurserna på den givna servern. Resurser för en server grupperas under [SKU](https://docs.microsoft.com/en-us/partner-center/develop/product-resources#sku) eller prissättning. För att skala upp en instans så gör man det genom att välja en annan app service plan i azure, när man gör det så ser man vad det kommer kosta och vad för hårdvara som ingår. Det finns en gräns på hur mycket man kan skala upp, när man kommit nära den så kan det bara bättre att skala horisontalt istället. När man skalar ner så reduceras resurserna till serven. Exemple på att skala upp en applikation kan vara att öka RAM eller processorkraft för den virtuella maskinen som serven körs på.

![Vertikal skalning](https://i1.wp.com/thecodeblogger.com/wp-content/uploads/2020/06/azure-scale-up.png?resize=1200%2C800&ssl=1)
[Bild ifrån thecodeblogger.com](https://i1.wp.com/thecodeblogger.com/wp-content/uploads/2020/06/azure-scale-up.png?resize=1200%2C800&ssl=1)

### Skala horisontalt

När man skalar en applikation horisontalt eller skalar ut som det också kallas så ökar man antalet servrar (eller instanser av servrar) som används för att köra applikationen. Varje server har samma konfigurationer så alla servrar är likadana. Vertikal skalning har en gräns på hur mycket det går att skala up eftersom det finns en gräns på hur mycket RAM och processor kraft en dator kan ha. Det är då horisontell skalning är väldigt bra att använda istället. När man pratar om att skala in så innebär det att man minskar antaler servrar. Om man har en applikation som är kör på en server och ska skala ut den så kör man applikationen på 2 eller mer identiska servrar istället för en server.

![Horisontell skalning](https://i0.wp.com/thecodeblogger.com/wp-content/uploads/2020/06/azure-scale-out.png?resize=1200%2C800&ssl=1)
[Bild ifrån thecodeblogger.com](https://i0.wp.com/thecodeblogger.com/wp-content/uploads/2020/06/azure-scale-out.png?resize=1200%2C800&ssl=1)

## Prisskillnader

### Vertikal skalning

Om man ska skala en app service vertikalt så varierar prist lite olika om man uppgraderar Tier. Kör man på Standard tier (som kostar 73 Dollar) och ska öka till Premium V2 (som kostar 146 Dollar) så blir priset det dubbla men om man har Premium V2 och ökar till Premium V3 (som kostar 244,55 Dollar) så ökar månadskostnaden med unefär 67%. Om man bara vill uppgradera i samma Tier så blir det dubbelt så dyrt för varje gång man uppgraderar. Kör man med Standard Tier så kostar det först 73 Dollar per månad och uppgraderar man RAM och processor kraft en gång så blir det dubbelt så dyrt som utgångs priset. Uppgraderar man RAM och processor kraften två gånger så blir det fyra gånger så dyrt som utgångs priset.

Ska man skala en virutell maskin vertikalt så är det samma prinsip om man håller sig till samma serie av virutell maskin. Det finns olika typer av serier istället för Tiers och om man ska använda en serie så borde man kolla upp hur mycket RAM och processor kraft en serien kan ha som max så man slipper byta till en annan serie.

### Horisontell skalning

När man skalar en app service horisontellt så betalar man för varje instans man har. Så om man har en app service med Standard Tier så kostar det 73 Dollar för en instans och om man vill skala till 3 instanser så kommer det kosta 219 Dollar eftersom man betalar 73 Dollar för varje instans. Ska man skala upp en viruell manskin så är det samma prinsip som när man skalar en app service horisontellt.

## App service plans

* Free, går bara att skala horisontalt
* shared, går bara att skala horisontalt
* Basic, går att skala vertikalt och horisontalt
* Standard, går att skala vertikalt och horisontalt
* Premium V2, går att skala vertikalt och horisontalt
* Premium V3, går att skala vertikalt och horisontalt
* Isolated, går att skala vertikalt och horisontalt
* Isolated V2, går att skala vertikalt och horisontalt

## Referenser

[Azure Pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

[You are currently viewing Understanding vertical and horizontal scaling in Azure
Understanding vertical and horizontal scaling in Azure](https://thecodeblogger.com/2020/06/30/understanding-vertical-and-horizontal-scaling-in-azure/)

[SKU](https://docs.microsoft.com/en-us/partner-center/develop/product-resources#sku)

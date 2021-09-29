---
layout: post
title: Närverk i målnet
subtitle: Uppgift 7
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, Nätverk]
---

Kära CTO jag tycker du ska tänka om över ditt beslut om molnlagring och vår nuvarande nätverk. Det kan vara riskabelt att lägga upp produkter i molnet men inte om  man använder sig av säkerhetsprinciper som också är lätta att sätta upp.

Vi kan använda oss utav Azure Private Link och Azure Service Bus för att kunna skicka data säkert. Med Azure private link så får man Privat åtkomst till tjänster på Azure plattformen via ett VPC (Virtual Private Cloud). Ett Virtual Private Cloud är ett säkert, isolerat privat moln som finns i ett offentligt moln. Azure Private Link ger även Skydd mot dataläckage och ger en global räckvidd genom att ansluta privat till tjänster som körs i andra regioner. Azure Service Bus används för att koppla bort applikationer och tjänster från varandra. Det ger följande fördelar som Lastbalansering, säker routing och överföring av data och kontroll över service och applikationsgränser och Koordinering av transaktionsarbete som kräver hög grad av tillförlitlighet. Med bara dem två tjänsterna kan man bygga ett säkert och stabilt grund för attbygga ut nätverket i framtiden. 

Här under kommer lite mer och bättre information om Azure Private Link och Azure service bus.


### Azure Private Link

Med Azure Private Link kan man komma åt Azure PaaS Services (till exempel Azure Storage och SQL Databaser ) och Azure hosted customer-owned/partner services via en privat endpoint i sitt virtuella nätverk. Trafik mellan ditt virtuella nätverk och tjänsten skickas i Microsoft backbone network. Vilket gör att man inte behöver skicka någon data öppet. Några fördelar med Azure private Link är:

..* Privat åtkomst till tjänster på Azure plattformen: Anslut vårat virtuella nätverk till tjänster i Azure utan en offentlig IP adress vid sourcen eller destinationen. Private Link -plattformen hanterar anslutningen mellan konsument och tjänster över Azure backbone network.
..* Lokala och peered-nätverk: Åtkomst till tjänster som körs i Azure från lokala via ExpressRoute privat peering, VPN-tunnlar och virtuella nätverk med privata slutpunkter. Private Link ger ett säkert sätt att migrera arbetsbelastningar till Azure.
..* Skydd mot dataläckage: En privat slutpunkt mappas till en instans av en PaaS -resurs istället för hela tjänsten. Konsumenter kan bara ansluta till den specifika resursen. Åtkomst till andra resurser i tjänsten är blockerad. Denna mekanism ger skydd mot dataläckage.
..* Global räckvidd: Anslut privat till tjänster som körs i andra regioner. Konsumentens virtuella nätverk kan vara i region A och det kan ansluta till tjänster bakom Private Link i region B.

### Azure service bus

Microsoft Azure Service Bus är en fullt hanterad företagsmeddelandemäklare med meddelandeköer och publicera-prenumerera ämnen. Service Bus används för att koppla bort applikationer och tjänster från varandra, vilket ger följande fördelar:
..* Lastbalansering
..* Säker routing och överföring av data och kontroll över service och applikationsgränser
..* Koordinering av transaktionsarbete som kräver hög grad av tillförlitlighet



Hoppas jag kunnat övertyga dig till att tänka om.

MVH Kristian Jimmefors

### Referenser

[Azure Private Link](https://docs.microsoft.com/en-us/azure/private-link/private-link-overview)
[Azure service bus](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)


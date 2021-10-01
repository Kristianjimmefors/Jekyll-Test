---
layout: post
title: Filer i målnet
subtitle: Uppgift 8
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, Molnet]
---

## Filer i molnet

Jag har gjort en konsolapplikation som kan ladda upp filer till [Azure Blob Storage](https://azure.microsoft.com/sv-se/services/storage/blobs/). Man lägger filen i en mapp och ändrar en variabel till namnet på filen som man lade till i mappen. Här är ett diagram som visar hur datan flyter i applikationen: 

![Dataflow]()


* ```BlobServiceClient``` klassen tillåter en att manipulera Azure storage resources och blob containrar.
* ```BlobContainerClient``` klassen tillåter en att manipulera Azure storage containers och deras blobar.
* ```BlobClient```klassen tillåter en att manipulera Azure storage blobs.

Först så har jag gjort en enviroment variabel som innehåller connection strängen till min storage container i Azure. För att komma åt containern så görs en ny instans av ```BlobServiceClient``` klassen där connection strängen till Azure storage containern skickas med. Efter det så används ```BlobContainerClient``` för att komma åt kontainern som ska användas och i detta fall så är det en kontainer som heter "bildtest" som jag skapade i Azure innan jag gjorde applikationen.
```c#
class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Azure Blob Storage .NET Sample");
            string connectionString = Environment.GetEnvironmentVariable("BLOBCON_STRING");

            BlobServiceClient blobServClient = new BlobServiceClient(connectionString);
            BlobContainerClient containerClient = blobServClient.GetBlobContainerClient("bildtest");
```

Filerna som ska laddas upp ligger i en mapp som heter data. För att kunna ladda upp filer till Azure så skapas en ny instans ```BlobClient``` för att sätta ett namn på vad filen ska heta uppe i Azure. Efter det så skrivs länken ut till filen som laddas upp. Innan det tar och laddar upp så kollas det om filen går att öppna. Om det går att öpna filen så laddas den upp till Azure med namnet som man har angett i ```fileName``` variabeln.
```c#
            string fileName = "image.png";

            string localFilePath = Path.Combine(Environment.CurrentDirectory, @"data\", fileName);
            BlobClient blobClient = containerClient.GetBlobClient(fileName);

            try
            {
                Console.WriteLine($"Uploading Image to Storage:\n\t{blobClient.Uri}\n");

                using FileStream uploadFileStream = File.OpenRead(localFilePath);

                await blobClient.UploadAsync(uploadFileStream);
                uploadFileStream.Close();

                Console.WriteLine("File uploaded successfully");
            }
            catch (Exception ex)
            {

                throw ex;
            }
            
        }
    }

```

## Pris för köra en stor applikation

Med de inställningar som bilden visar så skulle det kosta 968 Dollar eller ca 8498 Kronor i månaden att driva applikationen och spara bilder. Om man skulle ha 1000 användare som laddar upp 100 MB data varje dag och alla bilder som finns sparade laddas ner tre gånger per dag så skulle priset skena iväg väldigt mycket. Eftersom att det laddas upp nästan 100 GB per dag så blir priset för lagringen väldigt dyr och lagringen står för nästan allt i priset. Detta exemple som bilden visar skulle bara klara att lagra data som laddats upp i ett år efter det så behövs det mer lagringsutrymme.  Om man har drivit applikationen i 5 år och sparat alla bilder så skulle det kosta 4370 Dollar och ca 38364 Kronor i månaden för att bara ha lagringsutrymmet för att ha sparat allt.

![Cost of running big applikation]()

## Säkerhet Blob storage

När datan inte förflyttar sig så är den krypterad. När datan skrivs till Blob storage så är det en symmetrisk krypteringsnyckel som krypterar datan och det är olika nycklar som används för partionerad data. Nyckel för att dekrypterea datan är säkert lagrad med åtkomstkontrollistor som begränsar åtkomsten till nycklarna. Nyckelanvändningen loggas även vilket gör att man kan se när och vart dem använts. När data skickas över internet så är all data som skickas med Azure tjänster är säkrade med hjälp av Transport Layer Secutory (TLS) kryptografiskt protokoll och Forward Security Layer (även känt som Perfect Forward Secrecy eller PFS).


## Referenser
[Azure Blob Storage using a .NET Core Console Application](https://medium.com/@rammonzito/azure-blob-storage-using-a-net-core-console-application-106a0c2e6de5)

[how does azure encrypt data](https://cloudacademy.com/blog/how-does-azure-encrypt-data/)

[Pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

[Convert bytes](https://convertlive.com/u/convert/bytes/to/megabytes)

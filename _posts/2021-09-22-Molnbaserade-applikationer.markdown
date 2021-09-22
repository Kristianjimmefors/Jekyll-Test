---
layout: post
title: Serverless applikationer
subtitle: Uppgift 4
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, serverless]
---

## TODO applikation

Jag har gjort en simple todo lista där man kan läggat till, ta bort och visa alla eller en sak i listan.

För koden följde jag denna guide [Creating simple CRUD API](https://markheath.net/post/azure-functions-rest-csharp-bindings) för det mesta. Koden använder sig utav cosmos db som databas.

Klassen som jag anävnder har två properties som har standard värden annars får man tilldela dem andra värden själv.

```C#
 public class Todo
    {
        public string Id { get; set; }
        public DateTime Created { get; set; } = DateTime.Now;
        public string Name { get; set; }
        public string Description { get; set; }
        public bool IsCompleted { get; set; } = false;
        public string Category { get; set; }
    }
```

Här är koden för att lägga till en ny sak i todo listan och här har jag gjort ett litet fult sätt för att skapa ett id som jag lägger till istället för ett auto genererat. Det finns även en backup här i koden ifall ```input``` variablen blir null.
```C#
[FunctionName("CreateTodo")]
        public static async Task<IActionResult> CreateTodo(
            [HttpTrigger(AuthorizationLevel.Admin, "post", Route = "todo")] HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoList",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection")]
            IAsyncCollector<object> todos, ILogger log)
        {
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var input = JsonConvert.DeserializeObject<Todo>(requestBody);

            var todo = new Todo() { Id = req.Query["id"], Name = req.Query["name"], Description = req.Query["description"] };
            if (input != null)
            {
                todo = new Todo() { Description = input.Description, Name = input.Name };
            }

            await todos.AddAsync(new { id = todo.Id, todo.Name, todo.Description, todo.Created, todo.IsCompleted, todo.PartitionKey });

            return new OkObjectResult(todo);
        }
```

För att hämta ut ett item ifrån databasen så görs det lätt med ```CosmosDB(databaseName: "ToDoList", collectionName: "Items", ConnectionStringSetting = "CosmosDBConnection", Id ="{id}", PartitionKey ="{category}")``` vilket gör att man inte behöver skriva någon sql querry. ```CosmosDBConnection``` reffererar till en connection string som ligger i local.settings.json filen.
```C#
        [FunctionName("GetTaskById")]
        public static IActionResult GetTaskById(
            [HttpTrigger(AuthorizationLevel.Admin, "get", Route = "todo/{id}/{category}")] HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoList",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
            Id ="{id}",
            PartitionKey ="{category}")] Todo todo,
            ILogger log, string id, string category)
        {

            if (todo == null)
            {
                log.LogInformation("Item {id} was not found");

                return new NotFoundObjectResult(id);
            }

            return new OkObjectResult(todo);

        }
```

För att få fram alla items ifrån databasen så kör jag queryn i Cosmos DB triggern och exporterar resultatet av queryn som körs till ```IEnumerable<Todo> todos``` som sedan returneras i ett okej objekt resultat.
```C#
        [FunctionName("GetAllTasks")]
        public static IActionResult GetAllTasks(
        [HttpTrigger(AuthorizationLevel.Admin, "get", Route = "todo")] HttpRequest req,
        [CosmosDB(
                    databaseName: "ToDoList",
                    collectionName: "Items",
                    ConnectionStringSetting = "CosmosDBConnection",
                    SqlQuery = "SELECT * FROM c WHERE c.IsCompleted = false order by c._ts desc")]
                    IEnumerable<Todo> todos
                , ILogger log)
        {
            return new OkObjectResult(todos);

        }
```
jag tog och skapade en uri som länkar till databasen och collectionen som allt ligger i och efter det körs en query där ett id väljs ut.För att ta bort ett item ur databasen så behövde jag enabla cross partition query.
```C#
        [FunctionName("DeleteTodo")]
        public static async Task<IActionResult> DeleteTodo(
            [HttpTrigger(AuthorizationLevel.Admin, "delete", Route = "todo/{id}/{category}")] HttpRequest req,
            [CosmosDB(
                    ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            ILogger log, string id, string category)
        {
            Uri collectionUri = UriFactory.CreateDocumentCollectionUri("ToDoList", "Items");

            //enable cross partition querry
            var options = new FeedOptions { EnableCrossPartitionQuery = true };
            var document = client.CreateDocumentQuery(collectionUri, options).Where( x => x.Id == id).AsEnumerable().FirstOrDefault();
            
            if (document == null)
            {
                return new NotFoundObjectResult("Object do not exist");
            }

            log.LogInformation("Deleting todo item {id}");

            await client.DeleteDocumentAsync(document.SelfLink, new RequestOptions { PartitionKey = new Microsoft.Azure.Documents.PartitionKey(category) });
            return new OkObjectResult("Deleting todo item {id}");
        }
```

## Azure Cosmos DB

jag använder Azure Cosmos DB serverless och i den så finns det en databas som heter ToDo och den har en container som heter Tasks. Den har en Partition key som heter Category så man vet vilken kategori som ens Task tillhör.

![Cosmos DB bild](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/CosmosDB.PNG)

För att få igång databasen så gjorde jag bara en ny resurs av Cosmos DB, valde serverless  och följde de andra instruktionerna för att skapa instansen och göra en ny databas. Uppdatering av databasen är inte något som jag har tänkt så mycket på eftersom att det är ett väldigt simpelt projekt och databasen behöver inte vara mer avanserad just nu. Om man behöver så kan man lägga till en ny container om man tycker att det behövs, databasen är väldigt simple och behöver inte bli mer avancerad om man inte bygger ut applikationen mer. Databasen är väldigt simple så man behöver inte tänka på så mycket om eller runt den.

## Driftkostnad

Om man inte har så många användare så kanske detta skulle passa och då kostar det 59 Dollar eller ca 510 Kr i månaden.
![Small Cosmos DB Price](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/Small-Price-CosmosDB.PNG)

Om man har väldigt många användare såkanske detta skulle passa och då kostar det 9550 Dollar eller ca 82777 Kr i månaden
![Big Cosmos DB Price](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/Big-Price-CosmosDB.PNG)

## Referenser

[Creating simple CRUD API](https://markheath.net/post/azure-functions-rest-csharp-bindings)

[Azure Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

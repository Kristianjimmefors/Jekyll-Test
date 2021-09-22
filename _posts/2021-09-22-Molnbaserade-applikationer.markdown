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

För koden följde jag denna guide [Creating simple CRUD API](https://markheath.net/post/azure-functions-rest-csharp-bindings) för det mesta.

Här är koden för att lägga till en ny sak i todo listan och här har jag gjort ett litet fult sätt för att skapa ett id som jag lägger till istället för ett auto genererat.
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
            log.LogInformation("Creating todo");
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


```C#
        [FunctionName("GetTaskById")]
        public static IActionResult GetTaskById(
            [HttpTrigger(AuthorizationLevel.Admin, "get", Route = "todo/{id}")] HttpRequest req,
            [CosmosDB(
                databaseName: "ToDoList",
                collectionName: "Items",
                ConnectionStringSetting = "CosmosDBConnection",
            Id ="{id}",
            PartitionKey ="Todo")] Todo todo,
            ILogger log, string id)
        {

            log.LogInformation("Geting todo with " + id);

            if (todo == null)
            {
                log.LogInformation("Item {id} was not found");

                return new NotFoundObjectResult(id);
            }

            return new OkObjectResult(todo);

        }
```

```C#
        [FunctionName("GetAllTasks")]
        public static IActionResult GetAllTasks(
        [HttpTrigger(AuthorizationLevel.Admin, "get", Route = "todo")] HttpRequest req,
        [CosmosDB(
                    databaseName: "ToDoList",
                    collectionName: "Items",
                    ConnectionStringSetting = "CosmosDBConnection",
                    SqlQuery = "SELECT * FROM c order by c._ts desc")]
                    IEnumerable<Todo> todos
                , ILogger log)
        {
            log.LogInformation("Geting todos");

            return new OkObjectResult(todos);

        }
```

```C#
        [FunctionName("DeleteTodo")]
        public static async Task<IActionResult> DeleteTodo(
            [HttpTrigger(AuthorizationLevel.Admin, "delete", Route = "todo/{id}")] HttpRequest req,
            [CosmosDB(
                    databaseName: "ToDoList",
                    collectionName: "Items",
                    ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
            ILogger log, string id)
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

            await client.DeleteDocumentAsync(document.SelfLink, new RequestOptions { PartitionKey = new Microsoft.Azure.Documents.PartitionKey("Todo") });
            return new OkObjectResult("Deleting todo item {id}");

        }
```

## Referenser
[Creating simple CRUD API](https://markheath.net/post/azure-functions-rest-csharp-bindings)

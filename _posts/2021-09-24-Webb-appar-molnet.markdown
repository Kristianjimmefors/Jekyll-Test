---
layout: post
title: Webappar i målnet
subtitle: Uppgift 6
cover-img: /assets/img/cloud-show.jpg
thumbnail-img: /assets/img/cloud-show.jpg
share-img: /assets/img/cloud-show.jpg
categories: [Internet, måln]
---

## TODO applikation

Jag har gjort en simple todo lista där man kan läggat till och visa alla "tasks". Applikationen är en Razor Pages applikation och databasen är en Cosmos DB som ligger i azure

För koden så tog jag hjälp utav [Develop an ASP.NET Core with Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-dotnet-application#create-a-new-mvc-application)

Detta är det som behövs för att applikationen ska kunna komunisera med databasen i Azure. ```InitializeCosmosClientInstanceAsync``` metoden innehåller all nödvändig information så som namn på  databasen, container namn och partion key path. Den inehåller även account name och account key för cosmos databasen i azure.
```c#
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb")).GetAwaiter().GetResult());
        }

        //Creates a Cosmos DB database and a container with the specified partition key.
        private static async Task<CosmosDbService> InitializeCosmosClientInstanceAsync(IConfigurationSection configurationSection)
        {
            string databaseName = configurationSection.GetSection("DatabaseName").Value;
            string containerName = configurationSection.GetSection("ContainerName").Value;
            string account = configurationSection.GetSection("Account").Value;
            string key = configurationSection.GetSection("Key").Value;
            Microsoft.Azure.Cosmos.CosmosClient client = new Microsoft.Azure.Cosmos.CosmosClient(account, key);
            CosmosDbService cosmosDbService = new CosmosDbService(client, databaseName, containerName);
            Microsoft.Azure.Cosmos.DatabaseResponse database = await client.CreateDatabaseIfNotExistsAsync(databaseName);
            await database.Database.CreateContainerIfNotExistsAsync(containerName, "/Category");

            return cosmosDbService;
        }
```

I ```CosmosDbService``` klassen ligger alla metoder för som komunicerar med databasen så som att hämta all data och lägga till ny. Den extendar ett interface för metoderna.
```c#
public class CosmosDbService : ICosmosDbService
    {
        private Container _container;

        public CosmosDbService(
            CosmosClient dbClient,
            string databaseName,
            string containerName)
        {
            _container = dbClient.GetContainer(databaseName, containerName);
        }

        public async Task AddItemAsync(TaskItem item)
        {
            try
            {
                await _container.CreateItemAsync<TaskItem>(item, new PartitionKey(item.Category));

            }
            catch (Exception ex)
            {

                throw ex;
            }
        }

        public async Task DeleteItemAsync(TaskItem item,string id)
        {
            try
            {
                await _container.DeleteItemAsync<TaskItem>(id, new PartitionKey(item.Category));

            }
            catch (Exception ex)
            {

                throw ex;
            }
        }

        public async Task<List<TaskItem>> GetNotCompletedItems()
        {
            List<TaskItem> items = new List<TaskItem>();
            try
            {
                QueryDefinition query = new QueryDefinition("SELECT * FROM c WHERE c.IsCompleted = false ORDER BY c.Created ASC");

                var iterator = _container.GetItemQueryIterator<TaskItem>(query);

                while (iterator.HasMoreResults)
                {
                    FeedResponse<TaskItem> result = await iterator.ReadNextAsync();
                    foreach (var item in result)
                    {
                        items.Add(item);
                    }
                }
            }
            catch (CosmosException ex) when (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                return null;
            }
            return items;

        }

        public async Task UpdateItemAsync(string id, TaskItem item)
        {
            try
            {
                await _container.UpsertItemAsync(item, new PartitionKey(item.Category));

            }
            catch (Exception ex)
            {

                throw ex;
            }
        }
    }
```

Så här ser interfacet ut.
```c#
public interface ICosmosDbService
    {
        Task<List<TaskItem>> GetNotCompletedItems();
        Task AddItemAsync(TaskItem item);
        Task UpdateItemAsync(string id, TaskItem item);
        Task DeleteItemAsync(TaskItem item, string id);
    }
```
Detta är klassen som används för applikationen och databasen, den har ett standard värde på Created så att det inte missas att skickas med.
```c#
public class TaskItem
    {
        [JsonProperty(PropertyName = "id")]
        public string Id { get; set; }

        [JsonProperty(PropertyName = "name")]
        public string Name { get; set; }

        [JsonProperty(PropertyName = "description")]
        public string Description { get; set; }

        [JsonProperty(PropertyName = "Category")]
        public string Category { get; set; }

        [JsonProperty(PropertyName = "iscompleted")]
        public bool IsCompleted { get; set; } = false;

        [JsonProperty(PropertyName = "created")]
        public DateTime Created { get; set; } = DateTime.Now;
    }
```

Här hämtars värdena ut som postats i ett formulär när man gör ett nytt "task" och läggs in i ett nytt TaskItem klassen. Efter det så körs metoden för att spara det i databasen.
```c#
    public class NewTaskModel : PageModel
    {
        private readonly ILogger<NewTaskModel> _logger;
        private readonly ICosmosDbService _cosmosDbService;
        public NewTaskModel(ILogger<NewTaskModel> logger, ICosmosDbService cosmosDbService)
        {
            _logger = logger;
            _cosmosDbService = cosmosDbService;
        }
        public void OnGet()
        {
        }

        public void OnPost()
        {
            TaskItem newItem = new TaskItem
            {
                Name = Request.Form["name"].ToString(),
                Description = Request.Form["description"].ToString(),
                Category = Request.Form["category"].ToString()
            };

            _cosmosDbService.AddItemAsync(newItem);
        }
    }
```

## Köra appen i azure

För att få applikationen att köra i azure så pushade jag upp den till Azure container regestry via Visual studio eftersom den körs i docker. Efter det så skapade jag en ny Azure App Service via Visual Studio och när det var gjort så pushade jag applikationen dit. Först så gjock det inte att göra detta eftersom att containern i Azure Container Regestry inte hade admin user aktiverat. Efter det var aktiverat så funkade det att pusha applikationen till min app service.
![Pusha via Visual Studio](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/Push-to-asure.PNG)


## Prisskillnader

Om man ska använda sig av Cosmos DB och Azure App Service med en liten applikation så kan det kosta 114 Dollar eller ca 985 Kr i månaden.
![Low budget](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/Small-app-cosmos.PNG)

Det kan kosta 10 658 Dollar eller ca 92030 Kr om man har väldigt många användare.
![Big budget](https://raw.githubusercontent.com/Kristianjimmefors/Programmerings-grottan/main/assets/img/Big-app-cosmos.PNG)

## Referenser

[Develop an ASP.NET Core with Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-dotnet-application#create-a-new-mvc-application)

# Serverless and Schema-less with Azure Functions and Azure Cosmos DB
This repo contains the presentation slides and code samples for serverless and schema-less with Azure Functions and Azure Cosmos DB

# Local coding instructions

For purposes of convenience, this demo and code samples are geared to running on a local development machine with minimal external dependencies. 

### Prerequisites

* Visual Studio 2017 with Azure Development Workload
* Install Azure storage emulator [download](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator)
* Install CosmosDB emulator [download](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)
* Install latest version of Azure Functions tools for v2 [check out v2.0 section](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)

###### Nuget Packages: 
* Microsoft.Azure.WebJobs.Extensions.CosmosDB (latest version)

###### Before you start:
- Make sure Azure Storage emulator is running 
- Make sure CosmosDB emulator is running

###### Http Trigger
- Create a new Azure Functions project in visual studio. Make sure you pick v2 as the engine not v1.
- Add an empty project with no functions
- Add a new function, call it HttpTriggerFunction, pick Http Trigger and anonymous access (for this demo).
- In the function code file, changed function name attribute to CheckoutEndPoint or something else that makes sense
- Run your code and hit up the endpoint at the url generated 

###### Add Cosmos DB Http Trigger
- Add Nuget package: Microsoft.Azure.WebJobs.Extensions.CosmosDB (latest version)

- Paste the connection string for the CosmosDB emulator into your local.settings.json file

```json
{
    "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "CosmosDBConnection": "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
  }
}
```

- Replace the function code body with this chunk
```csharp
        [FunctionName("CheckoutEndPoint")]
        public static void Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]HttpRequest req,
            [CosmosDB(databaseName: "salesdb",
            collectionName: "salescollection",
            CreateIfNotExists = true,
            CollectionThroughput = 1000,
            PartitionKey = "/sku",
            ConnectionStringSetting = "CosmosDBConnection")]
        out dynamic outputDocument,
            ILogger log)
        {
            string requestBody = (new StreamReader(req.Body).ReadToEndAsync()).Result;
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            outputDocument = data;
        }
```

- In above code sample: the "CosmosDB" attribute with "out" indicates outgoing data. By deserializing the request and assigning it to the "out" variable we're sending it to the Cosmos DB datastore indicate in the "databaseName", "collectionName" and "ConnectionString" attributes. The CreateIfNotExists=true is a defensive coding mechanism that creates a database and/or collection if they don't exist.
 
- When you run this sample, the database and collection will autocreate if they don't exist. 

- Copy-paste the json data in the file in your favorite REST client body and issue a POST to the CheckoutEndPoint. You will notice that the data gets stored in the "salescollection" data store - verify with local emulator Data Explorer.



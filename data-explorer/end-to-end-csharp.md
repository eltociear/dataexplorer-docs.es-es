---
title: Ingesta de blobs de un extremo a otro en Azure Data Explorer a través de C#
description: En este artículo, aprenderá a ingerir blobs en Azure Data Explorer con un ejemplo de un extremo a otro que usa C#.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/19/2020
ms.openlocfilehash: 53504f51f1a5bd80967c510bd4bff1ba47cd8e58
ms.sourcegitcommit: ee90472a4f9d751d4049744d30e5082029c1b8fa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/21/2020
ms.locfileid: "83722174"
---
# <a name="end-to-end-blob-ingestion-into-azure-data-explorer-through-c"></a>Ingesta de blobs de un extremo a otro en Azure Data Explorer a través de C#

> [!div class="op_single_selector"]
> * [C#](end-to-end-csharp.md)
> * [Python](end-to-end-python.md)
>

Azure Data Explorer es un servicio de exploración de datos escalable y rápido para datos de telemetría y registro. En este artículo se ofrece un ejemplo completo de cómo ingerir datos de Azure Blob Storage en Azure Data Explorer. 

Aprenderá a crear un grupo de recursos mediante programación, una cuenta de almacenamiento y un contenedor, un centro de eventos y un clúster y una base de datos de Azure Data Explorer. También aprenderá a configurar Azure Data Explorer mediante programación para ingerir datos de la nueva cuenta de almacenamiento.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="install-c-nuget"></a>Instalación de NuGet C#

* Instale [Microsoft.Azure.Management.kusto](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/).
* Instale [Microsoft.Azure.Management.ResourceManager](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager).
* Instale [Microsoft.Azure.Management.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.Management.EventGrid/).
* Instale [Microsoft.Azure.Storage.Blob](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/).
* Instale [Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication) para llevar a cabo la autenticación.

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

[!INCLUDE [data-explorer-e2e-event-grid-resource-template](includes/data-explorer-e2e-event-grid-resource-template.md)]

## <a name="code-example"></a>Ejemplo de código 

En el ejemplo de código siguiente se proporciona un proceso paso a paso que da como resultado la ingesta de datos en Azure Data Explorer. 

Primero se crea un grupo de recursos. También se crean los recursos de Azure, como una cuenta de almacenamiento y un contenedor, un centro de eventos y un clúster y una base de datos de Azure Data Explorer, y se agregan entidades de seguridad. A continuación, crea una suscripción de Azure Event Grid, junto con una asignación de tabla y columna, en la base de datos de Azure Data Explorer. Finalmente, crea la conexión de datos para configurar Azure Data Explorer para ingerir datos de la nueva cuenta de almacenamiento. 

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
string location = "West Europe";
string locationSmallCase = "westeurope";
string azureResourceTemplatePath = @"xxxxxxxxx\template.json";//Path to the Azure Resource Manager template JSON from the previous section

string deploymentName = "e2eexample";
string resourceGroupName = deploymentName + "resourcegroup";
string eventHubName = deploymentName + "eventhub";
string eventHubNamespaceName = eventHubName + "ns";
string storageAccountName = deploymentName + "storage";
string storageContainerName = deploymentName + "storagecontainer";
string eventGridSubscriptionName = deploymentName + "eventgrid";
string kustoClusterName = deploymentName + "kustocluster";
string kustoDatabaseName = deploymentName + "kustodatabase";
string kustoTableName = "Events";
string kustoColumnMappingName = "Events_CSV_Mapping";
string kustoDataConnectionName = deploymentName + "kustoeventgridconnection";

//principals
string principalIdForCluster = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
string roleForClusterPrincipal = "AllDatabasesAdmin";
string tenantIdForClusterPrincipal = tenantId;
string principalTypeForCluster = "App";
string principalIdForDatabase = "xxxxxxxx@xxxxxxxx.com";//User Email
string roleForDatabasePrincipal = "Admin";
string tenantIdForDatabasePrincipal = tenantId;
string principalTypeForDatabase = "User";

var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, clientSecret);
var resourceManagementClient = new ResourceManagementClient(serviceCreds);
Console.WriteLine("Step 1: Create a new resource group in your Azure subscription to manage all the resources for using Azure Data Explorer.");
resourceManagementClient.SubscriptionId = subscriptionId;
await resourceManagementClient.ResourceGroups.CreateOrUpdateAsync(resourceGroupName,
    new ResourceGroup() { Location = locationSmallCase });

Console.WriteLine(
    "Step 2: Create a Blob Storage, a container in the Storage account, an Event Hub, an Azure Data Explorer cluster, database, and add principals by using an Azure Resource Manager template.");
var parameters = new Dictionary<string, Dictionary<string, object>>();
parameters["eventHubNamespaceName"] = new Dictionary<string, object>(capacity: 1) {{"value", eventHubNamespaceName}};
parameters["eventHubName"] = new Dictionary<string, object>(capacity: 1) {{"value", eventHubName }};
parameters["storageAccountName"] = new Dictionary<string, object>(capacity: 1) {{"value", storageAccountName }};
parameters["containerName"] = new Dictionary<string, object>(capacity: 1) {{"value", storageContainerName }};
parameters["kustoClusterName"] = new Dictionary<string, object>(capacity: 1) {{"value", kustoClusterName }};
parameters["kustoDatabaseName"] = new Dictionary<string, object>(capacity: 1) {{"value", kustoDatabaseName }};
parameters["principalIdForCluster"] = new Dictionary<string, object>(capacity: 1) {{"value", principalIdForCluster }};
parameters["roleForClusterPrincipal"] = new Dictionary<string, object>(capacity: 1) {{"value", roleForClusterPrincipal }};
parameters["tenantIdForClusterPrincipal"] = new Dictionary<string, object>(capacity: 1) {{"value", tenantIdForClusterPrincipal }};
parameters["principalTypeForCluster"] = new Dictionary<string, object>(capacity: 1) {{"value", principalTypeForCluster }};
parameters["principalIdForDatabase"] = new Dictionary<string, object>(capacity: 1) {{"value", principalIdForDatabase }};
parameters["roleForDatabasePrincipal"] = new Dictionary<string, object>(capacity: 1) {{"value", roleForDatabasePrincipal }};
parameters["tenantIdForDatabasePrincipal"] = new Dictionary<string, object>(capacity: 1) {{"value", tenantIdForDatabasePrincipal }};
parameters["principalTypeForDatabase"] = new Dictionary<string, object>(capacity: 1) {{"value", principalTypeForDatabase }};
            
string template = File.ReadAllText(azureResourceTemplatePath, Encoding.UTF8);
await resourceManagementClient.Deployments.CreateOrUpdateAsync(resourceGroupName, deploymentName,
    new Deployment(new DeploymentProperties(DeploymentMode.Incremental, template: template,
        parameters: parameters)));

Console.WriteLine(
    "Step 3: Create an Event Grid subscription to publish blob events created in a specific container to an Event Hub.");
var eventGridClient = new EventGridManagementClient(serviceCreds)
{
    SubscriptionId = subscriptionId
};
string storageResourceId = $"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}";
string eventHubResourceId = $"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.EventHub/namespaces/{eventHubNamespaceName}/eventhubs/{eventHubName}";
await eventGridClient.EventSubscriptions.CreateOrUpdateAsync(storageResourceId, eventGridSubscriptionName,
    new EventSubscription()
    {
        Destination = new EventHubEventSubscriptionDestination(eventHubResourceId),
        Filter = new EventSubscriptionFilter()
        {
            SubjectBeginsWith = $"/blobServices/default/containers/{storageContainerName}",
            IncludedEventTypes = new List<string>(){"Microsoft.Storage.BlobCreated"}
        }
    });

Console.WriteLine("Step 4: Create a table (with three columns: EventTime, EventId, and EventSummary) and column mapping in your Azure Data Explorer database.");
var kustoUri = $"https://{kustoClusterName}.{locationSmallCase}.kusto.windows.net";
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(kustoUri)
{
    InitialCatalog = kustoDatabaseName,
    FederatedSecurity = true,
    ApplicationClientId = clientId,
    ApplicationKey = clientSecret,
    Authority = tenantId
};

using (var kustoClient = KustoClientFactory.CreateCslAdminProvider(kustoConnectionStringBuilder))
{
    var command =
        CslCommandGenerator.GenerateTableCreateCommand(
            kustoTableName,
            new[]
            {
                Tuple.Create("EventTime", "System.DateTime"),
                Tuple.Create("EventId", "System.Int32"),
                Tuple.Create("EventSummary", "System.String"),
            });

    kustoClient.ExecuteControlCommand(command);

    command = CslCommandGenerator.GenerateTableMappingCreateCommand(
        Data.Ingestion.IngestionMappingKind.Csv,
        kustoTableName,
        kustoColumnMappingName,
        new ColumnMapping[] {
            new ColumnMapping() { ColumnName = "EventTime", ColumnType = "dateTime", Properties = new Dictionary<string, string>() { { MappingConsts.Ordinal, "0" } } },
            new ColumnMapping() { ColumnName = "EventId", ColumnType = "int", Properties = new Dictionary<string, string>() { { MappingConsts.Ordinal, "1" } } },
            new ColumnMapping() { ColumnName = "EventSummary", ColumnType = "string", Properties = new Dictionary<string, string>() { { MappingConsts.Ordinal, "2" } } },
        });
    kustoClient.ExecuteControlCommand(command);
}

Console.WriteLine("Step 5: Add an Event Grid data connection. Azure Data Explorer will automatically ingest the data when new blobs are created.");
var kustoManagementClient = new KustoManagementClient(serviceCreds)
{
    SubscriptionId = subscriptionId
};
await kustoManagementClient.DataConnections.CreateOrUpdateAsync(resourceGroupName, kustoClusterName,
                kustoDatabaseName, dataConnectionName: kustoDataConnectionName, new EventGridDataConnection(storageResourceId, eventHubResourceId, consumerGroup: "$Default", location: location, tableName:kustoTableName, mappingRuleName: kustoColumnMappingName, dataFormat: "csv"));

```
| **Configuración** | **Descripción del campo** |
|---|---|---|
| tenantId | El identificador de inquilino. También se conoce como identificador de directorio.|
| subscriptionId | Identificador de suscripción que se usa para la creación de recursos.|
| clientId | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| clientSecret | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |

## <a name="test-the-code-example"></a>Prueba del ejemplo de código

1. Cargue un archivo en la cuenta de almacenamiento.

    ```csharp
    string storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=xxxxxxxxxxxxxx;AccountKey=xxxxxxxxxxxxxx;EndpointSuffix=core.windows.net";
    var cloudStorageAccount = CloudStorageAccount.Parse(storageConnectionString);
    CloudBlobClient blobClient = cloudStorageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference(storageContainerName);
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("test.csv");
    var blobContent = @"2007-01-01 00:00:00.0000000,2592,Several trees down
    2007-01-01 00:00:00.0000000,4171,Winter Storm";
    await blockBlob.UploadTextAsync(blobContent);
    ```
    |**Configuración** | **Descripción del campo**|
    |---|---|---|
    | storageConnectionString | La cadena de conexión de la cuenta de almacenamiento creada mediante programación.|

2. Ejecute una consulta de prueba en Azure Data Explorer.

    ```csharp
    var kustoUri = $"https://{kustoClusterName}.{locationSmallCase}.kusto.windows.net";
    var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(kustoUri)
    {
        InitialCatalog = kustoDatabaseName,
        FederatedSecurity = true,
        ApplicationClientId = clientId,
        ApplicationKey = clientSecret,
        Authority = tenantId
    };
    using (var kustoClient = KustoClientFactory.CreateCslQueryProvider(kustoConnectionStringBuilder))
    {
        var query = $"{kustoTableName} | take 10";
        using (var reader = kustoClient.ExecuteQuery(query) as DataTableReader2)
        {// Print the contents of each of the result sets. 
            while (reader.Read())
            {
                Console.WriteLine($"{reader[0]}, {reader[1]}, {reader[2]}");
            }
        }
    }
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Utilice el comando siguiente para eliminar el grupo de recursos y limpiar los recursos:

```csharp
await resourceManagementClient.ResourceGroups.DeleteAsync(resourceGroupName);
```

## <a name="next-steps"></a>Pasos siguientes

*  Para más información sobre otras formas de crear un clúster y una base de datos, consulte [Creación de un clúster y una base de datos de Azure Data Explorer mediante C#](create-cluster-database-csharp.md).
* Para más información sobre los métodos de ingesta, consulte [Ingesta de datos de Azure Data Explorer](ingest-data-overview.md).
* Para información acerca de la aplicación web, consulte [Inicio rápido: Consulta de datos en la interfaz de usuario web de Azure Data Explorer](web-query-data.md).
* [Escriba consultas](write-queries.md) con el lenguaje de consulta Kusto.

---
title: Creación de una conexión de datos de Event Grid para Azure Data Explorer con C#
description: En este artículo, aprenderá a crear una conexión de datos de Event Grid para Azure Data Explorer con C#.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 10/07/2019
ms.openlocfilehash: d2432d302640d20bb199972bc686dadab41278f1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493096"
---
# <a name="create-an-event-grid-data-connection-for-azure-data-explorer-by-using-c"></a>Creación de una conexión de datos de Event Grid para Azure Data Explorer con C#

> [!div class="op_single_selector"]
> * [Portal](ingest-data-event-grid.md)
> * [C#](data-connection-event-grid-csharp.md)
> * [Python](data-connection-event-grid-python.md)
> * [Plantilla de Azure Resource Manager](data-connection-event-grid-resource-manager.md)


El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Azure Data Explorer permite ingerir (cargar) datos procedentes de Event Hubs, IoT Hubs y blobs escritos en contenedores de blobs. En este artículo, va a crear una conexión de datos de Event Grid para Azure Data Explorer con C#.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene Visual Studio 2019 instalado, puede descargar y usar la versión **gratis** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Cree [un clúster y una base de datos](create-cluster-database-csharp.md).
* Cree [una tabla y una asignación de columnas](net-standard-ingest-data.md#create-a-table-on-your-test-cluster).
* Establezca [la base de datos y las directivas de tabla](database-table-policies-csharp.md) (opcional).
* Cree una [cuenta de almacenamiento con una suscripción de Event Grid](ingest-data-event-grid.md#create-an-event-grid-subscription-in-your-storage-account).

[!INCLUDE [data-explorer-data-connection-install-nuget-csharp](includes/data-explorer-data-connection-install-nuget-csharp.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-event-grid-data-connection"></a>Incorporación de una conexión de datos de Event Grid

En el ejemplo siguiente, se explica cómo se agrega una conexión de datos de Event Grid mediante programación. Consulte [Creación de una conexión de datos de Event Grid en Azure Data Explorer](ingest-data-event-grid.md#create-an-event-grid-data-connection-in-azure-data-explorer) para agregar una conexión de datos Event Grid en Azure Portal.

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
var authenticationContext = new AuthenticationContext($"https://login.windows.net/{tenantId}");
var credential = new ClientCredential(clientId, clientSecret);
var result = await authenticationContext.AcquireTokenAsync(resource: "https://management.core.windows.net/", clientCredential: credential);

var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);

var kustoManagementClient = new KustoManagementClient(credentials)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the Prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
var dataConnectionName = "myeventhubconnect";
//The event hub and storage account that are created as part of the Prerequisites
var eventHubResourceId = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.EventHub/namespaces/xxxxxx/eventhubs/xxxxxx";
var storageAccountResourceId = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.Storage/storageAccounts/xxxxxx";
var consumerGroup = "$Default";
var location = "Central US";
//The table and column mapping are created as part of the Prerequisites
var tableName = "StormEvents";
var mappingRuleName = "StormEvents_CSV_Mapping";
var dataFormat = DataFormat.CSV;

await kustoManagementClient.DataConnections.CreateOrUpdateAsync(resourceGroupName, clusterName, databaseName, dataConnectionName,
    new EventGridDataConnection(storageAccountResourceId, eventHubResourceId, consumerGroup, tableName: tableName, location: location, mappingRuleName: mappingRuleName, dataFormat: dataFormat));
```

|**Configuración** | **Valor sugerido** | **Descripción del campo**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino. También conocido como identificador de directorio.|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador de suscripción que se usa para la creación de recursos.|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| clientSecret | *xxxxxxxxxxxxxx* | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |
| resourceGroupName | *testrg* | Nombre del grupo de recursos que contiene el clúster.|
| clusterName | *mykustocluster* | Nombre del clúster.|
| databaseName | *mykustodatabase* | Nombre de la base de datos de destino del clúster.|
| dataConnectionName | *myeventhubconnect* | Nombre que desea asignar a la conexión de datos.|
| tableName | *StormEvents* | Nombre de la tabla de destino de la base de datos de destino.|
| mappingRuleName | *StormEvents_CSV_Mapping* | Nombre de la asignación de columnas asociada a la tabla de destino.|
| dataFormat | *csv* | Formato de datos del mensaje.|
| eventHubResourceId | *Identificador del recurso* | Identificador del recurso del centro de eventos en el que se ha configurado Event Grid para que envíe eventos. |
| storageAccountResourceId | *Identificador del recurso* | Identificador del recurso de la cuenta de almacenamiento que contiene los datos que se van a ingerir. |
| consumerGroup | *$Default* | Grupo de consumidores del centro de eventos.|
| ubicación | *Centro de EE. UU.* | Ubicación del recurso de conexión de datos.|

## <a name="generate-sample-data"></a>Generación de datos de ejemplo

Ahora que Azure Data Explorer y la cuenta de almacenamiento están conectados, puede crear datos de ejemplo y cargarlos en Blob Storage.

Este script crea un contenedor en la cuenta de almacenamiento, carga un archivo existente (como un blob) en un contenedor y luego enumera los blobs en el contenedor.

```csharp
var azureStorageAccountConnectionString=<storage_account_connection_string>;

var containerName=<container_name>;
var blobName=<blob_name>;
var localFileName=<file_to_upload>;

// Creating the container
var azureStorageAccount = CloudStorageAccount.Parse(azureStorageAccountConnectionString);
var blobClient = azureStorageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference(containerName);
container.CreateIfNotExists();

// Set metadata and upload file to blob
var blob = container.GetBlockBlobReference(blobName);
blob.Metadata.Add("rawSizeBytes", "4096‬"); // the uncompressed size is 4096 bytes
blob.Metadata.Add("kustoIngestionMappingReference", "mapping_v2‬");
blob.UploadFromFile(localFileName);

// List blobs
var blobs = container.ListBlobs();
```

> [!NOTE]
> Azure Data Explorer no eliminará los blobs con posterioridad a la ingesta. Conserve los blobs de tres a cinco días mediante el [ciclo de vida de Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal) para administrar la eliminación de blobs.

[!INCLUDE [data-explorer-data-connection-clean-resources-csharp](includes/data-explorer-data-connection-clean-resources-csharp.md)]

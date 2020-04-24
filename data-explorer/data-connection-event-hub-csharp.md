---
title: Creación de una conexión de datos de Event Hubs para Azure Data Explorer con C#
description: En este artículo, aprenderá a crear una conexión de datos de Event Hubs para Azure Data Explorer con C#.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 10/07/2019
ms.openlocfilehash: 54f7541321b50ae4d3cf679d9224f70cb21a46ab
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493084"
---
# <a name="create-an-event-hub-data-connection-for-azure-data-explorer-by-using-c"></a>Creación de una conexión de datos de Event Hubs para Azure Data Explorer con C#

> [!div class="op_single_selector"]
> * [Portal](ingest-data-event-hub.md)
> * [C#](data-connection-event-hub-csharp.md)
> * [Python](data-connection-event-hub-python.md)
> * [Plantilla de Azure Resource Manager](data-connection-event-hub-resource-manager.md)

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Azure Data Explorer permite ingerir (cargar) datos procedentes de Event Hubs, IoT Hubs y blobs escritos en contenedores de blobs. En este artículo, va a crear una conexión de datos de Event Hubs para Azure Data Explorer con C#.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene Visual Studio 2019 instalado, puede descargar y usar la versión **gratis** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Cree [un clúster y una base de datos](create-cluster-database-csharp.md).
* Cree [una tabla y una asignación de columnas](net-standard-ingest-data.md#create-a-table-on-your-test-cluster).
* Establezca [la base de datos y las directivas de tabla](database-table-policies-csharp.md) (opcional).
* Cree un [centro de eventos con los datos que se van a ingerir](ingest-data-event-hub.md#create-an-event-hub). 

[!INCLUDE [data-explorer-data-connection-install-nuget-csharp](includes/data-explorer-data-connection-install-nuget-csharp.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-event-hub-data-connection"></a>Incorporación de una conexión de datos de Event Hubs

En el ejemplo siguiente, se explica cómo se agrega una conexión de datos de Event Hubs mediante programación. Consulte [Conexión a Event Hubs](ingest-data-event-hub.md#connect-to-the-event-hub) para agregar una conexión de datos de Event Hubs en Azure Portal.

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
//The event hub that is created as part of the Prerequisites
var eventHubResourceId = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.EventHub/namespaces/xxxxxx/eventhubs/xxxxxx";
var consumerGroup = "$Default";
var location = "Central US";
//The table and column mapping are created as part of the Prerequisites
var tableName = "StormEvents";
var mappingRuleName = "StormEvents_CSV_Mapping";
var dataFormat = DataFormat.CSV;
await kustoManagementClient.DataConnections.CreateOrUpdateAsync(resourceGroupName, clusterName, databaseName, dataConnectionName, 
    new EventHubDataConnection(eventHubResourceId, consumerGroup, location: location, tableName: tableName, mappingRuleName: mappingRuleName, dataFormat: dataFormat));
```

|**Configuración** | **Valor sugerido** | **Descripción del campo**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino. También conocido como identificador de directorio.|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador de suscripción que se usa para la creación de recursos.|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| clientSecret | *xxxxxxxxxxxxxx* | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| resourceGroupName | *testrg* | Nombre del grupo de recursos que contiene el clúster.|
| clusterName | *mykustocluster* | Nombre del clúster.|
| databaseName | *mykustodatabase* | Nombre de la base de datos de destino del clúster.|
| dataConnectionName | *myeventhubconnect* | Nombre que desea asignar a la conexión de datos.|
| tableName | *StormEvents* | Nombre de la tabla de destino de la base de datos de destino.|
| mappingRuleName | *StormEvents_CSV_Mapping* | Nombre de la asignación de columnas asociada a la tabla de destino.|
| dataFormat | *csv* | Formato de datos del mensaje.|
| eventHubResourceId | *Identificador del recurso* | Identificador del recurso del centro de eventos que contiene los datos que se van a ingerir. |
| consumerGroup | *$Default* | Grupo de consumidores del centro de eventos.|
| ubicación | *Centro de EE. UU.* | Ubicación del recurso de conexión de datos.|

[!INCLUDE [data-explorer-data-connection-clean-resources-csharp](includes/data-explorer-data-connection-clean-resources-csharp.md)]

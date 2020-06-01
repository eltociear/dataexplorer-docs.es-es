---
title: 'Introducción del almacenamiento mediante la suscripción a Event Grid: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la introducción del almacenamiento con Event Grid suscripción de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: e8c5642e59999c7a1bd547bfcb17cc18bf5d9e15
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84258052"
---
# <a name="ingest-from-storage-using-event-grid-subscription"></a>Ingesta desde almacenamiento mediante una suscripción a Event Grid

Azure Explorador de datos ofrece una ingesta continua de Azure Storage (BLOB Storage y ADLSv2) con [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/overview) suscripción para las notificaciones creadas por blobs y streaming de estas notificaciones a Kusto a través de un centro de eventos.

## <a name="data-format"></a>Formato de datos

* Los blobs pueden estar en cualquiera de los [formatos admitidos](../../../ingestion-supported-formats.md).
* Los blobs se pueden comprimir en cualquiera de las [compresiones admitidas](../../../ingestion-supported-formats.md#supported-data-compression-formats) .

> [!NOTE]
> Idealmente, el tamaño de datos sin comprimir original debe formar parte de los metadatos del BLOB.
> Si no se especifica el tamaño sin comprimir, Azure Explorador de datos lo calculará según el tamaño del archivo. Proporcione el tamaño de los datos originales estableciendo la `rawSizeBytes` [propiedad](#ingestion-properties) en los metadatos del BLOB en el tamaño de los datos **sin comprimir** en bytes.
> Tenga en cuenta que hay un límite de tamaño sin comprimir de ingesta por cada archivo de 4 GB.

## <a name="ingestion-properties"></a>Propiedades de la ingesta

Puede especificar [las propiedades de ingesta](../../../ingestion-properties.md) de la ingesta de blobs a través de los metadatos del BLOB.
Puede establecer las siguientes propiedades:

|Propiedad. | Descripción|
|---|---|
| rawSizeBytes | Tamaño de los datos sin formato (sin comprimir). En Avro/ORC/Parquet, es el tamaño antes de que se aplique la compresión específica del formato.|
| kustoTable |  Nombre de la tabla de destino existente. Invalida el valor de `Table` establecido en la hoja `Data Connection`. |
| kustoDataFormat |  Formato de datos. Invalida el valor de `Data format` establecido en la hoja `Data Connection`. |
| kustoIngestionMappingReference |  Nombre de la asignación de ingesta existente que se va a usar. Invalida el valor de `Column mapping` establecido en la hoja `Data Connection`.|
| kustoIgnoreFirstRecord | Si se establece en `true` , Azure explorador de datos omite la primera fila del BLOB. Úselo en datos con formato tabular (CSV, TSV o similar) para omitir los encabezados. |
| kustoExtentTags | Cadena que representa [etiquetas](../extents-overview.md#extent-tagging) que se adjuntarán a la extensión resultante. |
| kustoCreationTime |  Invalida [$IngestionTime](../../query/ingestiontimefunction.md?pivots=azuredataexplorer) para el blob, con el formato de una cadena ISO 8601. Se usa para la reposición. |

## <a name="events-routing"></a>Enrutamiento de eventos

Al configurar una conexión de almacenamiento de blobs en Azure Explorador de datos clúster, especifique las propiedades de la tabla de destino (nombre de tabla, formato de datos y asignación). Este es el enrutamiento predeterminado para los datos, también denominados `static routing` .
También puede especificar las propiedades de la tabla de destino para cada BLOB mediante metadatos de BLOB. Los datos se enrutarán dinámicamente según lo especificado por [las propiedades de ingesta](#ingestion-properties).

A continuación se facilita un ejemplo para establecer las propiedades de ingesta en los metadatos de BLOB antes de cargarlos. Los blobs se enrutan a tablas diferentes.

Consulte el código de [ejemplo](#generating-data) para obtener más información sobre cómo generar datos.

 ```csharp
// Blob is dynamically routed to table `Events`, ingested using `EventsMapping` data mapping
blob = container.GetBlockBlobReference(blobName2);
blob.Metadata.Add("rawSizeBytes", "4096‬"); // the uncompressed size is 4096 bytes
blob.Metadata.Add("kustoTable", "Events");
blob.Metadata.Add("kustoDataFormat", "json");
blob.Metadata.Add("kustoIngestionMappingReference", "EventsMapping");
blob.UploadFromFile(jsonCompressedLocalFileName);
```

## <a name="create-event-grid-subscription"></a>Creación de una suscripción de Event Grid

> [!Note]
> Para obtener el mejor rendimiento, cree todos los recursos en la misma región que el clúster de Explorador de datos de Azure.

### <a name="prerequisites"></a>Prerrequisitos

* [Crear una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account). 
  Event Grid suscripción de notificación se puede establecer en Azure Storage cuentas de tipo `BlobStorage` o `StorageV2` . 
  También se admite la habilitación de [Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction) .
* [Cree un centro de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-create).

### <a name="event-grid-subscription"></a>Suscripción a Event Grid

* Kusto seleccionado `Event Hub` como el tipo de punto de conexión, que se usa para transportar notificaciones de eventos de almacenamiento de blobs. `Event Grid schema`es el esquema seleccionado para las notificaciones. Tenga en cuenta que cada concentrador uniforme puede atender una conexión.
* La conexión de suscripción de almacenamiento de blobs controla las notificaciones de tipo `Microsoft.Storage.BlobCreated` . Asegúrese de seleccionarlo al crear la suscripción. Tenga en cuenta que se omiten otros tipos de notificaciones, si se seleccionan.
* Una suscripción puede notificar eventos de almacenamiento en un contenedor o más. Si desea realizar un seguimiento de los archivos de un contenedor específico, establezca los filtros para las notificaciones de la siguiente manera: al configurar una conexión, preste especial atención a los siguientes valores: 
   * **Subject Begins with** Filter es el prefijo *literal* del contenedor de blobs. Como el patrón aplicado es *Comienza con*, puede abarcar varios contenedores. No se permiten comodines.
     *Debe* establecerse de la siguiente manera: *`/blobServices/default/containers/<prefix>`* . Por ejemplo: */blobServices/default/containers/StormEvents-2020-*
   * El campo **Asunto termina con** es el sufijo *literal* del blob. No se permiten comodines. Resulta útil para filtrar las extensiones de archivo.

Encontrará un tutorial detallado en cómo [crear una suscripción de Event Grid en la guía de la cuenta de almacenamiento](../../../ingest-data-event-grid.md#create-an-event-grid-subscription-in-your-storage-account) .

### <a name="data-ingestion-connection-to-azure-data-explorer"></a>Conexión de ingesta de datos a Azure Explorador de datos

* A través de Azure Portal: [cree una conexión de datos Event Grid en Azure explorador de datos](../../../ingest-data-event-grid.md#create-an-event-grid-data-connection-in-azure-data-explorer).
* Uso del SDK de .NET de administración de Kusto: [Agregar una conexión de datos Event Grid](../../../data-connection-event-grid-csharp.md#add-an-event-grid-data-connection)
* Uso del SDK de Python de administración de Kusto: [Agregar una conexión de datos Event Grid](../../../data-connection-event-grid-python.md#add-an-event-grid-data-connection)
* Con plantilla de ARM: [Azure Resource Manager plantilla para agregar una conexión de datos Event Grid](../../../data-connection-event-grid-resource-manager.md#azure-resource-manager-template-for-adding-an-event-grid-data-connection)

### <a name="generating-data"></a>Generación de datos

> [!NOTE]
> * `BlockBlob`Se usa para generar datos. No se admite `AppendBlob`.
<!--> * Using ADLSv2 storage requires using `CreateFile` API for uploading blobs and flush at the end. 
    Kusto will get 2 notificatiopns: when blob is created and when stream is flushed. First notification arrives before the data is ready and ignored. Second notification is processed.-->

A continuación se facilita un ejemplo para crear un BLOB desde un archivo local, establecer las propiedades de ingesta en los metadatos del BLOB y cargarlos:

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

// Set ingestion properties in blob's metadata & uploading the blob
var blob = container.GetBlockBlobReference(blobName);
blob.Metadata.Add("rawSizeBytes", "4096‬"); // the uncompressed size is 4096 bytes
blob.Metadata.Add("kustoIgnoreFirstRecord", "true"); // First line of this csv file are headers
blob.UploadFromFile(csvCompressedLocalFileName);
```

## <a name="blob-lifecycle"></a>Ciclo de vida de BLOB

Azure Explorador de datos no eliminará los blobs posteriores a la ingesta, pero los conservará durante tres o cinco días. Use el [ciclo de vida de almacenamiento de blobs de Azure](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal) para administrar su eliminación de blobs.

## <a name="known-issues"></a>Problemas conocidos

Al usar Azure Explorador de datos para [exportar](../data-export/export-data-to-storage.md) los archivos que se usan para la ingesta de Event Grid, deben tenerse en cuentan los siguientes elementos: 
* Las notificaciones de Event Grid *no* se desencadenan si la cadena de conexión proporcionada al comando de exportación o la cadena de conexión proporcionada a una [tabla externa](../data-export/export-data-to-an-external-table.md) es una cadena de conexión en [formato ADLS Gen2](../../api/connection-strings/storage.md#azure-data-lake-store)(por ejemplo, `abfss://filesystem@accountname.dfs.core.windows.net` ), *pero la cuenta de almacenamiento no está habilitada para el espacio de nombres jerárquico*. 
 * Si la cuenta no está habilitada para el espacio de nombres jerárquico, la cadena de conexión debe utilizar el formato [BLOB Storage](../../api/connection-strings/storage.md#azure-storage-blob) (por ejemplo, `https://accountname.blob.core.windows.net` ). 
 * Tenga en cuenta que la exportación funcionará como se espera incluso al usar la cadena de conexión ADLS Gen2 en este caso, pero las notificaciones no se desencadenarán y, por tanto, Event Grid ingesta no funcionará. 
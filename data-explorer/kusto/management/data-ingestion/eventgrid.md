---
title: Ingerir desde el almacenamiento mediante la suscripción a Event Grid - Explorador de azure Data Explorer Microsoft Docs
description: En este artículo se describe ingerir desde el almacenamiento mediante la suscripción a Event Grid en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 682c39b11292c7e198dba46dad9b5bfa3b520c0f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521529"
---
# <a name="ingest-from-storage-using-event-grid-subscription"></a>Ingerir desde el almacenamiento mediante la suscripción a Event Grid

Azure Data Explorer ofrece ingesta continua desde Azure Storage (Blob Storage y ADLSv2) con suscripción a [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/overview) para notificaciones creadas por blobs y transmisión de estas notificaciones a Kusto a través de un centro de eventos.

## <a name="data-format"></a>Formato de datos

* Los blobs pueden estar en cualquiera de los [formatos compatibles.](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats)
* Los blobs se pueden comprimir en cualquiera de las [compresiones admitidas](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats#supported-data-compression-formats)

> [!NOTE]
> Idealmente, el tamaño de datos sin comprimir original debe formar parte de los metadatos del blob.
> Si no se especifica el tamaño sin comprimir, Azure Data Explorer lo estimará en función del tamaño del archivo. Proporcione el tamaño de `rawSizeBytes` datos original estableciendo la [propiedad](#ingestion-properties) en los metadatos del blob en el tamaño de datos **sin comprimir** en bytes.
> Tenga en cuenta que hay un límite de tamaño sin comprimir de ingesta por archivo de 4 GB.

## <a name="ingestion-properties"></a>Propiedades de la ingesta

Puede especificar [las propiedades Ingestion](https://docs.microsoft.com/azure/data-explorer/ingestion-properties) de la ingesta de blobs a través de los metadatos del blob.
Puede establecer las siguientes propiedades:

|Propiedad | Descripción|
|---|---|
| rawSizeBytes | Tamaño de los datos sin comprimir. Para Avro/ORC/Parquet, este es el tamaño antes de aplicar la compresión específica del formato.|
| kustoTable |  Nombre de la tabla de destino existente. Reemplaza el `Table` conjunto `Data Connection` de la hoja. |
| kustoDataFormat |  Formato de datos. Reemplaza el `Data format` conjunto `Data Connection` de la hoja. |
| kustoIngestionMappingReference |  Nombre de la asignación de ingesta existente que se va a utilizar. Reemplaza el `Column mapping` conjunto `Data Connection` de la hoja.|
| kustoIgnoreFirstRecord | Si se `true`establece en , Azure Data Explorer omite la primera fila del blob. Utilícelo en datos de formato tabular (CSV, TSV o similares) para omitir encabezados. |
| kustoExtentTags | Cadena que representa [las etiquetas](../extents-overview.md#extent-tagging) que se adjuntarán en la medida resultante. |
| kustoCreationTime |  Reemplaza [$IngestionTime](../../query/ingestiontimefunction.md?pivots=azuredataexplorer) para el blob, con formato de cadena ISO 8601. Utilícelo para rellenar. |

## <a name="events-routing"></a>Enrutamiento de eventos

Al configurar una conexión de almacenamiento de blobs en el clúster de Azure Data Explorer, especifique las propiedades de la tabla de destino (nombre de tabla, formato de datos y asignación). Este es el enrutamiento predeterminado para los `static routig`datos, también se denomina .
También puede especificar propiedades de tabla de destino para cada blob mediante metadatos de blob. Los datos se enrutarán dinámicamente según lo especificado por las propiedades de [ingesta](#ingestion-properties).

A continuación se muestra un ejemplo para establecer propiedades de ingesta en los metadatos del blob antes de cargarlos. Los blobs se enrutan a tablas diferentes.

Consulte el [código](#generating-data) de ejemplo para obtener más detalles sobre cómo generar datos.

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
> Para obtener el mejor rendimiento, cree todos los recursos de la misma región que el clúster de Azure Data Explorer.

### <a name="prerequisites"></a>Prerrequisitos

* [Crear una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account). 
  La suscripción de notificación de Event `BlobStorage` `StorageV2`Grid se puede establecer en Cuentas de Azure Storage de tipo o . 
  También se admite la habilitación de [Data Lake Storage Gen2.](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)
* [Cree un centro de eventos.](https://docs.microsoft.com/azure/event-hubs/event-hubs-create)

### <a name="event-grid-subscription"></a>Suscripción a Event Grid

* Kusto `Event Hub` seleccionado como el tipo endpoind, utilizado para transportar notificaciones de eventos de almacenamiento de blobs. `Event Grid schema`es el esquema seleccionado para las notificaciones. Tenga en cuenta que cada Even Hub puede servir una conexión.
* La conexión de suscripción de `Microsoft.Storage.BlobCreated`almacenamiento de blobs controla las notificaciones de tipo . Asegúrese de seleccionarlo al crear la suscripción. Tenga en cuenta que se omiten otros tipos de notificaciones, si se seleccionan.
* Una suscripción puede notificar eventos de almacenamiento en un contenedor o más. Si desea realizar un seguimiento de los archivos de un contenedor específico, establezca los filtros para las notificaciones de la siguiente manera: Al configurar una conexión, tenga en cuenta los siguientes valores: 
   * El filtro Subject Begins With es el prefijo *literal* del contenedor de **blobs.** Como el patrón aplicado es *Comienza con*, puede abarcar varios contenedores. No se permiten comodines.
     *Debe* establecerse de la *`/blobServices/default/containers/<prefix>`* siguiente manera: . Por ejemplo: */blobServices/default/containers/StormEvents-2020-*
   * El campo **Asunto termina con** es el sufijo *literal* del blob. No se permiten comodines. Es útil para filtrar extensiones de archivo.

Puede encontrar un guía detallado en la guía [Cómo crear una suscripción](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-grid#create-an-event-grid-subscription-in-your-storage-account) a Event Grid en la cuenta de almacenamiento.

### <a name="data-ingestion-connection-to-azure-data-explorer"></a>Conexión de ingesta de datos a Azure Data Explorer

* A través de Azure Portal: Crear una conexión de datos de [Event Grid en Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-grid#create-an-event-grid-data-connection-in-azure-data-explorer).
* Uso del SDK de .NET de administración de Kusto: agregar una conexión de [datos de Event Grid](https://docs.microsoft.com/azure/data-explorer/data-connection-event-grid-csharp#add-an-event-grid-data-connection)
* Uso del SDK de Python de administración de Kusto: Agregue una conexión de [datos de Event Grid](https://docs.microsoft.com/azure/data-explorer/data-connection-event-grid-python#add-an-event-grid-data-connection)
* Con plantilla ARM: [plantilla de Azure Resource Manager para agregar una conexión](https://docs.microsoft.com/azure/data-explorer/data-connection-event-grid-resource-manager#azure-resource-manager-template-for-adding-an-event-grid-data-connection) de datos de Event Grid

### <a name="generating-data"></a>Generación de datos

> [!NOTE]
> * Se `BlockBlob` utiliza para generar datos. No se admite `AppendBlob`.
<!--> * Using ADLSv2 storage requires using `CreateFile` API for uploading blobs and flush at the end. 
    Kusto will get 2 notificatiopns: when blob is created and when stream is flushed. First notification arrives before the data is ready and ignored. Second notification is processed.-->

A continuación se muestra un ejemplo para crear un blob a partir de un archivo local, establecer propiedades de ingesta en los metadatos del blob y cargarlo:

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

## <a name="blob-lifecycle"></a>Ciclo de vida de blobs

Azure Data Explorer no eliminará los blobs después de la ingesta, pero los conservará durante tres a cinco días. Use el ciclo de vida de [Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal) para administrar la eliminación de blobs.
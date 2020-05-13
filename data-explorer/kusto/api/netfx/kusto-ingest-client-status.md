---
title: 'Informes de estado de ingesta de Kusto. ingesta: Azure Explorador de datos'
description: En este artículo se describe Kusto. ingesta sobre el estado de ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 76ae07e2e7bdbb15900385b1e2feab0c9ff97d01
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373630"
---
# <a name="kustoingest-ingestion-status-reporting"></a>Informes de estado de ingesta de Kusto. Ingeri

En este artículo se explica cómo usar las características de [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) para realizar un seguimiento del estado de una solicitud de ingesta.

## <a name="description-classes"></a>Clases de Descripción

Estas clases de Descripción contienen detalles importantes sobre los datos de origen que se van a ingerir y deben usarse en la operación de ingesta. 

* SourceDescription
* DataReaderDescription
* StreamDescription
* FileDescription
* BlobDescription

Todas las clases se derivan de la clase abstracta `SourceDescription` y se usan para crear instancias de un identificador único para cada origen de datos. Cada identificador se utilizará para el seguimiento de estado y se mostrará en todos los informes, seguimientos y excepciones relacionados con la operación pertinente.

### <a name="class-sourcedescription"></a>Clase SourceDescription

Los grandes conjuntos de valores se dividirán en fragmentos de 1 GB y cada parte se inscribirá por separado. El mismo SourceId se aplicará a todas las operaciones de ingesta originadas en el mismo conjunto de DataSet.   

```csharp
public abstract class SourceDescription
{
    public Guid? SourceId { get; set; }
}
```

### <a name="class-datareaderdescription"></a>Clase DataReaderDescription

```csharp
public class DataReaderDescription : SourceDescription
{
    public  IDataReader DataReader { get; set; }
}
```

### <a name="class-streamdescription"></a>Clase StreamDescription

```csharp
public class StreamDescription : SourceDescription
{
    public Stream Stream { get; set; }
}
```

### <a name="class-filedescription"></a>Clase FileDescription

```csharp
public class FileDescription : SourceDescription
{
    public string FilePath { get; set; }
}
```

### <a name="class-blobdescription"></a>Clase BlobDescription

```csharp
public class BlobDescription : SourceDescription
{
    public string BlobUri { get; set; }
    // Setting the Blob size here is important, as this saves the client the trouble of probing the blob for size
    public long? BlobSize { get; set; }
}
```

## <a name="ingestion-result-representation"></a>Representación del resultado de la ingesta

### <a name="interface-ikustoingestionresult"></a>Interfaz IKustoIngestionResult

Esta interfaz captura el resultado de una sola operación de ingesta en cola y puede ser recuperada por `SourceId` .

```csharp
public interface IKustoIngestionResult
{
    // Retrieves the detailed ingestion status of the ingestion source with the given sourceId.
    IngestionStatus GetIngestionStatusBySourceId(Guid sourceId);

    // Retrieves the detailed ingestion status of all data ingestion operations into Kusto associated with this IKustoIngestionResult instance.
    IEnumerable<IngestionStatus> GetIngestionStatusCollection();
}
```

### <a name="class-ingestionstatus"></a>Clase IngestionStatus

IngestionStatus contiene un estado completo de una única operación de ingesta.

```csharp
public class IngestionStatus
{
    // The ingestion status returns from the service. Status remains 'Pending' during the ingestion process and
    // is updated by the service once the ingestion completes. When <see cref="IngestionReportMethod"/> is set to 'Queue' the ingestion status
    // will always be 'Queued' and the caller needs to query the report queues for ingestion status, as configured. To query statuses that were
    // reported to queue, see: <see href="https://docs.microsoft.com/azure/kusto/api/netfx/kusto-ingest-client-status#ingestion-status-in-azure-queue"/>.
    // When <see cref="IngestionReportMethod"/> is set to 'Table', call <see cref="IKustoIngestionResult.GetIngestionStatusBySourceId"/> or
    // <see cref="IKustoIngestionResult.GetIngestionStatusCollection"/> to retrieve the most recent ingestion status.
    public Status Status { get; set; }
    // A unique identifier representing the ingested source. Can be supplied during the ingestion execution.
    public Guid IngestionSourceId { get; set; }
    // The URI of the blob, potentially including the secret needed to access the blob.
    // This can be a filesystem URI (on-premises deployments only), or an Azure Blob Storage URI (including a SAS key or a semicolon followed by the account key)
    public string IngestionSourcePath { get; set; }
    // The name of the database holding the target table.
    public string Database { get; set; }
    // The name of the target table into which the data will be ingested.
    public string Table { get; set; }
    // The last updated time of the ingestion status.
    public DateTime UpdatedOn { get; set; }
    // The ingestion's operation Id.
    public Guid OperationId { get; set; }
    // The ingestion operation activity Id.
    public Guid ActivityId { get; set; }
    // In case of a failure - indicates the failure's error code.
    public IngestionErrorCode ErrorCode { get; set; }
    // In case of a failure - indicates the failure's status.
    public FailureStatus FailureStatus { get; set; }
    // In case of a failure - indicates the failure's details.
    public string Details { get; set; }
    // In case of a failure - indicates whether or not the failures originate from an Update Policy.
    public bool OriginatesFromUpdatePolicy { get; set; }
}
```

### <a name="status-enumeration"></a>Enumeración de estado

|Value              |Significado                                                                                     |Temporal/permanente
|-------------------|-----------------------------------------------------------------------------------------------------|---------|
|Pending            |El valor puede cambiar durante el transcurso de la ingesta, en función del resultado de la operación de ingesta. |Temporales|
|Correcto          |Los datos se han ingerido correctamente                                                              |Permanente| 
|Con error             |Error de ingesta                                                                                     |Permanente|
|En cola             |Los datos se han puesto en cola para la ingesta                                                               |Permanente|
|Omitido            |No se proporcionó ningún dato y se omitió la operación de introducción                                            |Permanente|
|PartiallySucceeded |Parte de los datos se ingesta correctamente, mientras que algunos no                                        |Permanente|

## <a name="tracking-ingestion-status-kustoqueuedingestclient"></a>Seguimiento del estado de ingesta (KustoQueuedIngestClient)

[IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) es un cliente de "incendio y olvidar". La operación de ingesta en el lado cliente finaliza mediante la publicación de un mensaje en una cola de Azure. Después del envío, se realiza el trabajo de cliente. Para la comodidad del usuario del cliente, KustoQueuedIngestClient proporciona un mecanismo para hacer un seguimiento del estado de la ingesta individual. Este mecanismo no está diseñado para el uso masivo en canalizaciones de ingesta de alto rendimiento. Este mecanismo es para la ingesta de precisión cuando la tasa es relativamente baja y los requisitos de seguimiento son estrictos.

> [!WARNING]
> Se debe evitar la activación de notificaciones positivas para cada solicitud de ingesta de grandes flujos de datos de gran volumen, ya que esto hace que se produzca una carga extrema en los recursos de xStore subyacentes, lo que podría provocar una mayor latencia de ingesta e incluso completar la falta de capacidad de respuesta del clúster.

Las siguientes propiedades (establecidas en [KustoQueuedIngestionProperties](kusto-ingest-client-reference.md#class-kustoqueuedingestionproperties)) controlan el nivel y el transporte de las notificaciones correctas o erróneas de ingesta.

### <a name="ingestionreportlevel-enumeration"></a>Enumeración IngestionReportLevel

```csharp
public enum IngestionReportLevel
{
    FailuresOnly = 0,
    None,
    FailuresAndSuccesses
}
```

### <a name="ingestionreportmethod-enumeration"></a>Enumeración IngestionReportMethod

```csharp
public enum IngestionReportMethod
{
    Queue = 0,
    Table,
    QueueAndTable
}
```

Para realizar un seguimiento del estado de la ingesta, proporcione lo siguiente al [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) en el que realiza la operación de introducción:
1.  Establezca `IngestionReportLevel` la propiedad en el nivel de informe requerido. Cualquiera de `FailuresOnly` los dos (que es el valor predeterminado) o `FailuresAndSuccesses` .
Cuando se establece en, no se `None` informará nada al final de la ingesta.
1.  Especifique `IngestionReportMethod`  -  `Queue` , `Table` o `both` .

Puede encontrar un ejemplo de uso en la página de [ejemplos de Kusto. ingesta](kusto-ingest-client-examples.md) .

### <a name="ingestion-status-in-the-azure-table"></a>Estado de ingesta en la tabla de Azure

La `IKustoIngestionResult` interfaz que se devuelve de cada operación de introducción contiene funciones que se pueden utilizar para consultar el estado de la ingesta.
Preste especial atención a la `Status` propiedad de los objetos devueltos `IngestionStatus` :
* `Pending`indica que el origen se ha puesto en cola para la ingesta y todavía se está actualizando. Usar la función de nuevo para consultar el estado del origen
* `Succeeded`indica que el origen se ha ingesta correctamente
* `Failed`indica que no se pudo ingerir el origen.

> [!NOTE]
> Al obtener un `Queued` Estado se indica que `IngestionReportMethod` se dejó en su valor predeterminado de ' Queue '. Se trata de un estado permanente y volver a invocar `GetIngestionStatusBySourceId` las `GetIngestionStatusCollection` funciones o, siempre dará como resultado el mismo estado "en cola".
> Para comprobar el estado de una ingesta en una tabla de Azure, antes de la ingesta, compruebe que la `IngestionReportMethod` propiedad de [KustoQueuedIngestionProperties](kusto-ingest-client-reference.md#class-kustoqueuedingestionproperties) está establecida en `Table` . Si también desea que se notifique el estado de ingesta en una cola, establezca el estado en `QueueAndTable` .

### <a name="ingestion-status-in-azure-queue"></a>Estado de ingesta en la cola de Azure

Los `IKustoIngestionResult` métodos solo son relevantes para comprobar el estado de una tabla de Azure. Para consultar los Estados que se han comunicado a una cola de Azure, use los siguientes métodos de [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) .

|Método                                  |Propósito     |
|----------------------------------------|------------|
|PeekTopIngestionFailures                |Método asincrónico que devuelve información sobre los errores de ingesta más antiguos que todavía no se han descartado debido al límite de mensajes solicitados. |
|GetAndDiscardTopIngestionFailures       |Método asincrónico que devuelve y descarta los errores de ingesta más antiguos que todavía no se han descartado debido al límite de mensajes solicitados. |
|GetAndDiscardTopIngestionSuccesses      |Método asincrónico que devuelve y descarta los éxitos de ingesta más antiguos que todavía no se han descartado debido al límite de mensajes solicitados. Este método solo es relevante si el `IngestionReportLevel` valor de está establecido en.`FailuresAndSuccesses` |

### <a name="ingestion-failures-retrieved-from-the-azure-queue"></a>Errores de ingesta recuperados de la cola de Azure

El `IngestionFailure` objeto que contiene información útil sobre el error representa los errores de ingesta.

|Propiedad                      |Significado     |
|------------------------------|------------|
|Tabla de & de base de datos              |La base de datos y los nombres de tabla deseados |
|IngestionSourcePath           |Ruta de acceso del BLOB ingestado. Contendrá el nombre de archivo original si se ingeri el archivo. Será aleatorio si se ha ingerido DataReader. |
|FailureStatus                 |`Permanent`(no se ejecutará ningún reintento), `Transient` (se ejecutará el reintento) o (se producirán `Exhausted` errores en varios reintentos) |
|OperationId & RootActivityId  |IDENTIFICADOR de operación e identificador de RootActivity de la ingesta (útil para más información) |
|Con errores                      |Hora UTC del error. Será mayor que la hora en que se llamó al método de ingesta, ya que los datos se agregan antes de ejecutar la ingesta. |
|Detalles                       |Otros detalles relativos al error (si existe) |
|ErrorCode                     |`IngestionErrorCode`enumeración, representa el código de error de ingesta, si se produjo un error |

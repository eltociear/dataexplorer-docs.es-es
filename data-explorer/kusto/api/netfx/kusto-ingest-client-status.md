---
title: Referencia de Kusto.Ingest - Informes de estado de ingesta - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Kusto.Ingest Reference - Inging Status Reporting en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 1a3eed1db0ec7d3dd4bc83c0a65f342020b2a387
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523722"
---
# <a name="kustoingest-reference---ingestion-status-reporting"></a>Referencia de Kusto.Ingest - Informe sin estado de la ingesta
En este artículo se explica cómo utilizar las capacidades [de IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) para realizar el seguimiento del estado de una solicitud de ingesta.

## <a name="sourcedescription-datareaderdescription-streamdescription-filedescription-and-blobdescription"></a>SourceDescription, DataReaderDescription, StreamDescription, FileDescription y BlobDescription
Estas diversas clases de descripción encapsulan detalles importantes con respecto a los datos de origen que se van a ingerir, e incluso pueden proporcionarse a la operación de ingesta.
Todas estas clases se derivan `SourceDescription` de la clase abstracta y se usan para crear instancias de un identificador único para cada origen de datos.
El identificador proporcionado está pensado para el seguimiento posterior del estado de la operación y se mostrará en todos los informes, seguimientos y excepciones relacionados con la operación adecuada.

### <a name="class-sourcedescription"></a>Clase SourceDescription
>Al ingericar un conjunto de datos grande (por ejemplo, un DataReader de más de 1 GB): los datos se dividirán en fragmentos de 1 GB y se ingerirán por separado.<BR>En este caso, el mismo SourceId se aplicará a todas las operaciones de ingesta originadas en el mismo conjunto de datos.   

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

## <a name="ingestion-result-representation"></a>Representación de resultados de ingestión

### <a name="interface-ikustoingestionresult"></a>Interfaz IKustoIngestionResult
Esta interfaz captura el resultado de una sola operación `SourceId`de ingesta en cola y permite su recuperación por .
```csharp
public interface IKustoIngestionResult
{
    // Retrieves the detailed ingestion status of the ingestion source with the given sourceId.
    IngestionStatus GetIngestionStatusBySourceId(Guid sourceId);

    // Retrieves the detailed ingestion status of all data ingestion operations into Kusto associated with this IKustoIngestionResult instance.
    IEnumerable<IngestionStatus> GetIngestionStatusCollection();
}
```

### <a name="class-ingestionstatus"></a>IngestionStatus de clase
IngestionStatus encapsula un estado completo de una sola operación de ingesta.
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
|Value |Significado |
|------------|------------|
|Pending |Temporal. Podría n.o cambiar durante el curso de la ingestión en función del resultado de la operación de ingestión |
|Correcto |Permanente. los datos han sido ingeridos con éxito |
|Con error |Permanente. Error en la ingestión |
|En cola |Permanente. Los datos se han puesto en cola para la ingestión |
|Omitido |Permanente. No se proporcionaron datos y se omitió la operación de ingesta |
|PartiallySucceeded |Permanente. Parte de los datos se ha ingerido con éxito, mientras que algunos |

## <a name="tracking-ingestion-status-kustoqueuedingestclient"></a>Estado de ingesta de seguimiento (KustoQueuedIngestClient)
[IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) es un cliente "fire-and-forget": la operación de ingesta en el lado cliente finaliza publicando un mensaje en una cola de Azure, después de lo cual se realiza el trabajo de cliente. Para la comodidad del usuario cliente, KustoQueuedIngestClient proporciona un mecanismo para realizar el seguimiento del estado de ingesta individual. Esto no está pensado para el uso masivo en tuberías de ingesta de alto rendimiento, sino más bien para la ingesta de "precisión" cuando la tasa es relativamente baja y los requisitos de seguimiento son muy estrictos.

> [!WARNING]
> Debe evitarse activar las notificaciones positivas para cada solicitud de ingesta de flujos de datos de gran volumen, ya que esto pone una carga extrema en los recursos xStore subyacentes, > lo que podría dar lugar a una mayor latencia de ingesta e incluso a una falta de respuesta completa del clúster.


Las siguientes propiedades (establecidas en [KustoQueuedIngestionProperties](kusto-ingest-client-reference.md#class-kustoqueuedingestionproperties)) controlan el nivel y el transporte para las notificaciones de éxito o error de ingesta:

### <a name="ingestionreportlevel-enumeration"></a>IngestionReportLevel enumeración
```csharp
public enum IngestionReportLevel
{
    FailuresOnly = 0,
    None,
    FailuresAndSuccesses
}
```

### <a name="ingestionreportmethod-enumeration"></a>IngestionReportMethod enumeración
```csharp
public enum IngestionReportMethod
{
    Queue = 0,
    Table,
    QueueAndTable
}
```

Para poder realizar un seguimiento del estado de la ingesta, asegúrese de proporcionar lo siguiente a [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) con la que realiza la operación de ingesta:
1.  Establecer `IngestionReportLevel`propiedad en el nivel deseado de informe: FailuresOnly (que es el valor predeterminado) o FailuresAndSuccesses.
Cuando se establece en Ninguno, no se notificará nada al final de la ingestión.
2.  Especifique el `IngestionReportMethod` deseado - Cola, Tabla o ambos.

Puede encontrar un ejemplo de uso en la página Ejemplos de [Kusto.Ingest.](kusto-ingest-client-examples.md)

### <a name="ingestion-status-in-azure-table"></a>Estado de ingesta en la tabla de Azure
La `IKustoIngestionResult` interfaz que se devuelve de cada operación de ingesta contiene funciones que se pueden usar para consultar el estado de la ingesta.
Preste especial atención `Status` a la `IngestionStatus` propiedad de los objetos devueltos:
* `Pending`indica que el origen se ha puesto en cola para la ingesta y aún no se ha actualizado; utilizar la función de nuevo para consultar el estado del origen
* `Succeeded`indica que la fuente se ha ingerido con éxito
* `Failed`indica que la fuente no pudo ser ingerida

>Tenga en cuenta `Queued` que obtener `IngestionReportMethod` un estado indica que se dejó en su valor predeterminado de 'Cola'. Se trata de un estado permanente y volver a invocar las funciones GetIngestionStatusBySourceId o GetIngestionStatusCollection siempre dará como resultado el mismo estado 'Queued'.<BR>Para poder comprobar el estado de una ingesta en una tabla de `IngestionReportMethod` Azure, compruebe antes de ingeniar `Table` que `QueueAndTable` la propiedad de [KustoQueuedIngestionProperties](kusto-ingest-client-reference.md#class-kustoqueuedingestionproperties) está establecida en (o si también desea que el estado de ingesta se notifique a una cola).

### <a name="ingestion-status-in-azure-queue"></a>Estado de ingesta en Azure Queue
Los `IKustoIngestionResult` métodos solo son relevantes para comprobar un estado en una tabla de Azure. Para consultar los estados notificados a una cola de Azure, utilice los métodos siguientes de [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient):

|Método |Propósito |
|------------|------------|
|PeekTopIngestionFailures |Método asincrónico que devuelve información sobre los errores de ingesta más antiguos que no se han descartado, según el límite de mensajes solicitados |
|GetAndDiscardTopIngestionFailures |Método asincrónico que devuelve y descarta los errores de ingesta más antiguos que no se han descartado, según el límite de mensajes solicitados |
|GetAndDiscardTopIngestionSuccesses |Método asincrónico que devuelve y descarta los primeros éxitos de ingesta que no se `IngestionReportLevel` han descartado, según el límite de mensajes solicitados (solo es relevante si se establece en`FailuresAndSuccesses` |


### <a name="ingestion-failures-retrieved-from-azure-queue"></a>Errores de ingesta recuperados de la cola de Azure
Los errores de ingesta `IngestionFailure` se representan mediante un objeto que contiene información útil sobre el error:

|Propiedad |Significado |
|------------|------------|
|Tabla de & de base de datos |La base de datos y los nombres de tabla previstos |
|IngestionSourcePath |La ruta de acceso del blob ingerido. Will contiene el nombre de archivo original en caso de ingesta del archivo. Será aleatorio en caso de ingestión de DataReader |
|FailureStatus |`Permanent`(no se ejecutará ningún `Transient` reintento), (se `Exhausted` ejecutará el reintento) o (también se han producido un error en varios reintentos) |
|OperationId & RootActivityId |ID de operación e ID de RootActivity de la ingesta (útil para solucionar problemas adicionales) |
|FailedOn |Hora UTC del error. Será mayor que el momento en que se llamó al método de ingestión, ya que los datos se agregan antes de ejecutar la ingesta |
|Detalles |Otros detalles relativos al fallo (si existe) |
|ErrorCode |`IngestionErrorCode`enumeración, que representa el código de error de ingesta, en caso de que exista el error|
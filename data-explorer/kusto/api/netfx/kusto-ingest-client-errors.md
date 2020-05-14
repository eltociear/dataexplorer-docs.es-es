---
title: 'Errores de Kusto. ingesta & excepciones: Azure Explorador de datos'
description: 'En este artículo se describe Kusto. ingesta: errores y excepciones en Azure Explorador de datos.'
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 97798fa62d588769636966c7155dc5f398bd001a
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382325"
---
# <a name="kustoingest-errors-and-exceptions"></a>Errores y excepciones de Kusto. ingesta
Cualquier error durante el control de ingesta en el lado cliente se indica mediante una excepción de C#.

## <a name="failures"></a>Errores

### <a name="kustodirectingestclient-exceptions"></a>Excepciones de KustoDirectIngestClient

Al intentar introducirse en varios orígenes, pueden producirse errores durante el proceso de ingesta. Si se produce un error en una ingesta para uno de los orígenes, se registra y el cliente continúa ingerindo el resto de orígenes. Después de pasar todos los orígenes para la ingesta, `IngestClientAggregateException` se produce una excepción que contiene el `IList<IngestClientException> IngestionErrors` miembro.

`IngestClientException`y sus clases derivadas contienen un campo `IngestionSource` y un `Error` campo. Los dos campos juntos crean una asignación, desde el origen en el que se produjo un error en la ingesta, hasta el error que se produjo al intentar la ingesta. La información se puede usar en la `IngestionErrors` lista para investigar qué orígenes ingesta con errores y por qué. La `IngestClientAggregateException` excepción también contiene una propiedad booleana `GlobalError` que indica si se produjo un error en todos los orígenes.

### <a name="failures-ingesting-from-files-or-blobs"></a>Ingesta de errores de archivos o BLOBs

Si se produce un error de ingesta al intentar la ingesta de un BLOB o un archivo, los orígenes de ingesta no se eliminarán, incluso si la `deleteSourceOnSuccess` marca está establecida en `true` . Los orígenes se conservan para su posterior análisis. Una vez que se entiende el origen del error, y si el error no se originó en el propio origen de ingesta, el usuario del cliente puede intentar reutilizarlo.

### <a name="failures-ingesting-from-idatareader"></a>Ingesta de errores de IDataReader

Durante la ingesta desde DataReader, los datos que se van a ingerir se guardan en una carpeta temporal cuya ubicación predeterminada es `<Temp Path>\Ingestions_<current date and time>` . Esta carpeta predeterminada siempre se elimina después de una ingesta correcta.

En los `IngestFromDataReader` `IngestFromDataReaderAsync` métodos y, la `retainCsvOnFailure` marca, cuyo valor predeterminado es `false` , determina si los archivos se deben mantener después de un error de ingesta. Si esta marca se establece en `false` , los datos que no cumplan la ingesta no se conservarán, por lo que es difícil entender qué salió mal.

## <a name="kustoqueuedingestclient-exceptions"></a>Excepciones de KustoQueuedIngestClient

`KustoQueuedIngestClient`ingeri datos mediante la carga de mensajes en una cola de Azure. Si se produce un error antes o durante el proceso de puesta en cola, `IngestClientAggregateException` se produce una excepción al final del proceso. La excepción iniciada incluye una colección de `IngestClientException` , que contiene el origen de cada error y que no se ha enviado a la cola. También se produce el error que se produjo al intentar publicar el mensaje.

### <a name="posting-to-queue-failures-with-a-file-or-blob-as-a-source"></a>Publicación en errores de cola con un archivo o BLOB como origen

Si se produce un error al utilizar `KustoQueuedIngestClient` los `IngestFromFile/IngestFromBlob` métodos de, no se eliminan los orígenes, aunque la `deleteSourceOnSuccess` marca esté establecida en `true` . En su lugar, los orígenes se conservan para su posterior análisis. 

Una vez comprendido el origen del error, y si el error no se originó en el propio origen de ingesta, el usuario del cliente puede intentar volver a poner en cola los datos mediante los `IngestFromFile/IngestFromBlob` métodos pertinentes con el origen con errores. 

### <a name="posting-to-queue-failures-with-idatareader-as-a-source"></a>Envío de errores de cola con IDataReader como origen

Al utilizar un origen DataReader, los datos que se van a publicar en la cola se guardan en una carpeta temporal cuya ubicación predeterminada es `<Temp Path>\Ingestions_<current date and time>` . Esta carpeta siempre se elimina una vez que los datos se han publicado correctamente en la cola.
En los `IngestFromDataReader` `IngestFromDataReaderAsync` métodos y, la `retainCsvOnFailure` marca, cuyo valor predeterminado es `false` , determina si los archivos se deben mantener después de un error de ingesta. Si esta marca se establece en `false` , los datos que no cumplan la ingesta no se conservarán, por lo que es difícil entender qué salió mal.

### <a name="common-failures"></a>Errores comunes
|Error                         |Motivo           |Mitigación                                   |
|------------------------------|-----------------|---------------------------------------------|
|El nombre de la base de datos <database name> no existe| La base de datos no existe|Compruebe el nombre de la base de datos en `kustoIngestionProperties` la base de datos/Create |
|No se encontró la entidad ' nombre de tabla que no existe ' de tipo ' tabla '.|La tabla no existe y no hay ninguna asignación de CSV.| Agregar asignación de CSV/crear la tabla necesaria |
|BLOB <blob path> excluido por razón: el patrón JSON se debe ingerir con el parámetro jsonMapping| Ingesta de JSON cuando no se proporciona ninguna asignación JSON.|Proporcionar una asignación JSON |
|No se pudo descargar el BLOB: "el servidor remoto devolvió un error: (404) no encontrado".| El blob no existe.|Compruebe que existe el BLOB. Si existe, vuelva a intentarlo y póngase en contacto con el equipo de Kusto |
|La asignación de columnas JSON no es válida: dos o más elementos de asignación apuntan a la misma columna.| La asignación JSON tiene 2 columnas con diferentes rutas de acceso|Corregir asignación JSON |
|EngineError-[UtilsException] `IngestionDownloader.Download` : no se pudieron descargar uno o varios archivos (busque KustoLogs para ActivityId <GUID1> :, RootActivityId: <GUID2> )| No se pudieron descargar uno o varios archivos. |Reintentar |
|No se pudo analizar: el flujo con ID. ' <stream name> ' tiene un formato CSV incorrecto, error por directiva de ValidationOptions |Archivo CSV con formato incorrecto (por ejemplo, no tiene el mismo número de columnas en cada línea). Solo produce un error cuando la Directiva de validación está establecida en `ValidationOptions` . ValidateCsvInputConstantColumns |Compruebe los archivos CSV. Este mensaje solo se aplica a los archivos CSV/TSV |
|`IngestClientAggregateException`con el mensaje de error "faltan los parámetros obligatorios para la firma de acceso compartido válida" |La SAS que se usa es del servicio y no de la cuenta de almacenamiento. |Usar la SAS de la cuenta de almacenamiento |

### <a name="ingestion-error-codes"></a>Códigos de error de ingesta

Para ayudar a controlar los errores de ingesta mediante programación, la información de errores se enriquece con un código de error numérico ( `IngestionErrorCode enumeration` ).

|ErrorCode                                      |Motivo                                                        |
|-----------------------------------------------|--------------------------------------------------------------|
|Desconocido                                        | Error desconocido|
|Stream_LowMemoryCondition                      | La operación se quedó sin memoria|
|Stream_WrongNumberOfFields                     | El documento CSV tiene un número de campos incoherente|
|Stream_InputStreamTooLarge                     | El documento enviado para la ingesta ha superado el tamaño permitido|
|Stream_NoDataToIngest                          | No se encontraron flujos de datos para ingerir|
|Stream_DynamicPropertyBagTooLarge              | Una de las columnas dinámicas de los datos ingeridos contiene demasiadas propiedades únicas.|
|Download_SourceNotFound                        | No se pudo descargar el origen de Azure Storage: no se encontró el origen|
|Download_AccessConditionNotSatisfied           | No se pudo descargar el origen de Azure Storage: acceso denegado|
|Download_Forbidden                             | No se pudo descargar el origen de Azure Storage: acceso no permitido|
|Download_AccountNotFound                       | No se pudo descargar el origen de Azure Storage: no se encontró la cuenta|
|Download_BadRequest                            | No se pudo descargar el origen de Azure Storage: solicitud incorrecta|
|Download_NotTransient                          | No se pudo descargar el origen de Azure Storage, no se ha producido un error transitorio|
|Download_UnknownError                          | No se pudo descargar el origen de Azure Storage: error desconocido|
|UpdatePolicy_QuerySchemaDoesNotMatchTableSchema| No se pudo invocar la Directiva de actualización. El esquema de consulta no coincide con el esquema de tabla|
|UpdatePolicy_FailedDescendantTransaction       | No se pudo invocar la Directiva de actualización. Directiva de actualización transaccional de descendientes con error|
|UpdatePolicy_IngestionError                    | No se pudo invocar la Directiva de actualización. Error de ingesta|
|UpdatePolicy_UnknownError                      | No se pudo invocar la Directiva de actualización. Error desconocido|
|BadRequest_MissingJsonMappingtFailure          | El patrón JSON no se ingesta con el parámetro jsonMapping|
|BadRequest_InvalidOrEmptyBlob                  | El BLOB no es válido o es un archivo zip vacío|
|BadRequest_DatabaseNotExist                    | La base de datos no existe|
|BadRequest_TableNotExist                       | La tabla no existe|
|BadRequest_InvalidKustoIdentityToken           | Token de identidad de Kusto no válido|
|BadRequest_UriMissingSas                       | Ruta de acceso de BLOB sin SAS de almacenamiento de blobs desconocido|
|BadRequest_FileTooLarge                        | Intentando ingerir un archivo demasiado grande|
|BadRequest_NoValidResponseFromEngine           | No hay ninguna respuesta válida del comando de introducción|
|BadRequest_TableAccessDenied                   | Se denegó el acceso a la tabla|
|BadRequest_MessageExhausted                    | El mensaje está agotado|
|General_BadRequest                             | Solicitud general incorrecta. Sugerencia de la ingesta en una base de datos o tabla inexistente|
|General_InternalServerError                    | Error interno del servidor|

## <a name="detailed-exceptions-reference"></a>Referencia detallada de excepciones

### <a name="cloudqueuesnotfoundexception"></a>CloudQueuesNotFoundException

Se genera cuando no se devolvieron colas desde el clúster de Administración de datos

Clase base: [Exception](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|Nombre del campo |Tipo     |Significado
|-----------|---------|------------------------------|
|Error      | String  | Error que se produjo al intentar recuperar las colas del DM
                            
Solo es relevante cuando se usa el [cliente de introducción de Kusto en cola](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient).
Durante el proceso de ingesta, se realizan varios intentos para recuperar las colas de Azure vinculadas al DM. Cuando se produce un error en estos intentos, se genera la excepción que contiene el motivo del error en el campo ' error '. Es posible que también se produzca una excepción interna en el campo ' InnerException '.


### <a name="cloudblobcontainersnotfoundexception"></a>CloudBlobContainersNotFoundException

Se genera cuando no se devuelve ningún contenedor de blobs desde el clúster de Administración de datos

Clase base: [Exception](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|Nombre del campo   |Tipo     |Significado       
|-------------|---------|------------------------------|
|KustoEndpoint| String  | Extremo del DM pertinente
                            
Solo es relevante cuando se usa el [cliente de introducción de Kusto en cola](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient).  
Al ingesta de orígenes que aún no están en un contenedor de Azure, como archivos, DataReader o Stream, los datos se cargan en un BLOB temporal para la ingesta. La excepción se produce cuando no se encuentra ningún contenedor en el que cargar los datos.

### <a name="duplicateingestionpropertyexception"></a>DuplicateIngestionPropertyException

Se genera cuando una propiedad de ingesta se configura más de una vez

Clase base: [Exception](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|Nombre del campo   |Tipo     |Significado       
|-------------|---------|------------------------------------|
|PropertyName | String  | Nombre de la propiedad duplicada.
                            
### <a name="postmessagetoqueuefailedexception"></a>PostMessageToQueueFailedException

Se genera cuando se produce un error al publicar un mensaje en la cola

Clase base: [Exception](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

|Nombre del campo   |Tipo     |Significado       
|-------------|---------|---------------------------------|
|QueueUri     | String  | El URI de la cola.
|Error        | String  | Mensaje de error que se generó al intentar publicar en la cola
                            
Solo es relevante cuando se usa el [cliente de introducción de Kusto en cola](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient).  
El cliente de ingesta en cola ingesta datos mediante la carga de un mensaje en la cola de Azure correspondiente. Si hay un error post, se produce la excepción. Contendrá el URI de la cola, el motivo del error en el campo ' error ' y, posiblemente, una excepción interna en el campo ' InnerException '.

### <a name="dataformatnotspecifiedexception"></a>DataFormatNotSpecifiedException

Se genera cuando se requiere un formato de datos pero no se especifica en`IngestionProperties`

Clase base: IngestClientException

Al ingesta de un flujo, se debe especificar un formato de datos en el [IngestionProperties](kusto-ingest-client-reference.md#class-kustoingestionproperties)para ingerir los datos correctamente. Esta excepción se produce cuando `IngestionProperties.Format` no se especifica el parámetro.

### <a name="invaliduriingestclientexception"></a>InvalidUriIngestClientException

Se genera cuando se envía un URI de BLOB no válido como origen de ingesta.

Clase base: IngestClientException

### <a name="compressfileingestclientexception"></a>CompressFileIngestClientException

Se genera cuando el cliente de ingesta no puede comprimir el archivo proporcionado para la ingesta

Clase base: IngestClientException

Los archivos se comprimen antes de su ingesta. La excepción se produce cuando se produce un error al intentar comprimir el archivo.

### <a name="uploadfiletotempblobingestclientexception"></a>UploadFileToTempBlobIngestClientException

Se genera cuando el cliente de ingesta no puede cargar el origen proporcionado para la ingesta en un BLOB temporal.

Clase base: IngestClientException

### <a name="sizelimitexceededingestclientexception"></a>SizeLimitExceededIngestClientException

Se genera cuando un origen de ingesta es demasiado grande

Clase base: IngestClientException

|Nombre del campo   |Tipo     |Significado       
|-------------|---------|-----------------------|
|Tamaño         | long    | Tamaño del origen de la ingesta
|MaxSize      | long    | El tamaño máximo permitido para la ingesta

Si un origen de ingesta supera el tamaño máximo de 4 GB, se produce la excepción. La validación del tamaño puede reemplazarse por la `IgnoreSizeLimit` marca de la [clase IngestionProperties](kusto-ingest-client-reference.md#class-kustoingestionproperties). Sin embargo, no se recomienda introducir [orígenes únicos superiores a 1 GB](about-kusto-ingest.md#ingestion-best-practices).

### <a name="uploadfiletotempblobingestclientexception"></a>UploadFileToTempBlobIngestClientException

Se genera cuando el cliente de ingesta no puede cargar el archivo proporcionado para la ingesta en un BLOB temporal.

Clase base: IngestClientException

### <a name="directingestclientexception"></a>DirectIngestClientException

Se genera cuando se produce un error general mientras se realiza una ingesta directa

Clase base: IngestClientException

### <a name="queuedingestclientexception"></a>QueuedIngestClientException

Se genera cuando se produce un error mientras se realiza una ingesta en cola

Clase base: IngestClientException

### <a name="ingestclientaggregateexception"></a>IngestClientAggregateException

Se genera cuando se producen uno o varios errores durante una ingesta.

Clase base: [AggregateException](https://msdn.microsoft.com/library/system.aggregateexception(v=vs.110).aspx)

|Nombre del campo      |Tipo                             |Significado       
|----------------|---------------------------------|-----------------------|
|IngestionErrors | IList<IngestClientException>    | Los errores que se producen al intentar la ingesta y los orígenes relacionados con ellos.
|IsGlobalError   | bool                            | Indica si la excepción se produjo para todos los orígenes

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los errores en código nativo, vea [errores en código nativo](../../concepts/errorsinnativecode.md).

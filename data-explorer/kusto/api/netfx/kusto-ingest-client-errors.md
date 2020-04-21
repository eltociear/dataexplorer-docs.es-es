---
title: Referencia de Kusto.Ingest - Errores y excepciones - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Kusto.Ingest Reference - Errors and Exceptions in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: f8f50322a79dea8890b4a4ad5eaa78f0b8fe3bc0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81524351"
---
# <a name="kustoingest-reference---errors-and-exceptions"></a>Referencia de Kusto.Ingest - Errores y excepciones
Cualquier error durante el control de ingesta en el lado del cliente se expone al código de usuario a través de una excepción de C.

## <a name="failures-overview"></a>Descripción general de los fallos

### <a name="kustodirectingestclient-exceptions"></a>Excepciones de KustoDirectIngestClient
Al intentar ingerir de varias fuentes, pueden producirse errores durante la ingestión de algunas de esas fuentes, mientras que otros pueden ingerirse correctamente. Si se produce un error en una ingesta para un origen determinado, se registra y el cliente continúa ingestando los orígenes restantes para la ingesta. Después de repasar todas `IngestClientAggregateException` las fuentes de ingestión, se produce un, que contiene un miembro `IList<IngestClientException> IngestionErrors`.
`IngestClientException`y sus clases derivadas `IngestionSource` contienen un campo y un `Error` campo que juntos forman una asignación desde el origen que no se pudo ingerir al error que se produjo al intentar ingerirlo. Se puede utilizar la información de la lista IngestionErrors para investigar qué orígenes no se han ingerido y por qué. `IngestClientAggregateException`excepción también contiene `GlobalError`una propiedad booleana , que indica si se ha producido un error para todos los orígenes.

### <a name="failures-ingesting-from-files-or-blobs"></a>Errores de renuncia a archivos o blobs 
Si se ha producido un error de ingesta al intentar ingerir desde blob-archivo, los orígenes de ingesta no se eliminan, incluso si la `deleteSourceOnSuccess` marca está establecida en `true`.
Las fuentes se conservan para su posterior análisis. Una vez que se entiende el origen del error y dado que el error no se originó desde el propio origen de la ingesta, el usuario del cliente puede intentar volver a ingesta.

### <a name="failures-ingesting-from-idatareader"></a>Errores de renuncia a IDataReader
Al ingerir desde DataReader, los datos que se ingestan `<Temp Path>\Ingestions_<current date and time>`se guardan en una carpeta temporal cuya ubicación predeterminada es . Esta carpeta siempre se elimina después de una ingesta correcta.<BR>
En `IngestFromDataReader` los `IngestFromDataReaderAsync` métodos `retainCsvOnFailure` y, la `false`marca, cuyo valor predeterminado es , determina si los archivos deben mantenerse después de una ingesta con errores. Si esta marca `false`se establece en , los datos que fallan la ingesta no se conservarán, lo que hace difícil entender qué salió mal.

## <a name="kustoqueuedingestclient-exceptions"></a>Excepciones de KustoQueuedIngestClient
KustoQueuedIngestClient ingesta datos cargando mensajes en una cola de Azure. Si se produce algún error antes `IngestClientAggregateException` y durante el proceso de colocación `IngestClientException` en cola, se produce un al final de la ejecución con una colección que contiene el origen que no se ha publicado en la cola (para cada error) y el error que se produjo al intentar publicar el mensaje.

### <a name="posting-to-queue-failures-with-file-or-blob-as-a-source"></a>Publicar errores en la cola con archivo o blob como origen
Si se ha producido un error al utilizar los métodos IngestFromFile/IngestFromBlob de KustoQueuedIngestClient, los orígenes no se eliminan, incluso si la `deleteSourceOnSuccess` marca se establece en `true`, sino que se conservan más bien para su posterior análisis. Una vez que se comprende el origen del error y se ha dado cuenta de que el error no se originó desde el propio origen, el usuario del cliente puede intentar volver a poner en cola los datos mediante los métodos IngestFromFile/IngestFromBlob pertinentes con el origen con errores. 

### <a name="posting-to-queue-failures-with-idatareader-as-a-source"></a>Publicar en errores de cola con IDataReader como origen
Al utilizar un origen de DataReader, los datos que se publicarán `<Temp Path>\Ingestions_<current date and time>`en la cola se guardan en una carpeta temporal cuya ubicación predeterminada es .
Esta carpeta siempre se elimina después de que los datos se hayan publicado correctamente en la cola.
En `IngestFromDataReader` los `IngestFromDataReaderAsync` métodos `retainCsvOnFailure` y, la `false`marca, cuyo valor predeterminado es , determina si los archivos deben mantenerse después de una ingesta con errores. Si esta marca `false`se establece en , los datos que fallan la ingesta no se conservarán, lo que hace difícil entender qué salió mal.

### <a name="common-failures"></a>Fracasos comunes
|Error|Motivo|Mitigación|
|------------------------------|----|------------|
|El <database name> nombre de la base de datos no existe| La base de datos no existe|Compruebe el nombre de la base de datos en kustoIngestionProperties/Create the database |
|No se encontró la entidad 'nombre de tabla que no existe' de tipo 'Tabla'.|La tabla no existe y no hay ninguna asignación CSV.| Agregar asignación CSV / crear la tabla requerida |
|Blob <blob path> excluido por motivos: el patrón json debe ingerse con el parámetro jsonMapping| Ingestión de Json cuando no se proporciona ninguna asignación de json.|Proporcionar una asignación JSON |
|No se pudo descargar el blob: 'El servidor remoto devolvió un error: (404) No encontrado.'| El blob no existe.|Compruebe que el blob existe, si existe un reintento y póngase en contacto con el equipo de Kusto |
|La asignación de columnas Json no es válida: dos o más elementos de asignación apuntan a la misma columna.| La asignación JSON tiene 2 columnas con diferentes rutas de acceso|Corregir la asignación JSON |
|EngineError - [UtilsException] IngestionDownloader.Download: No se han podido descargar uno<GUID1>o varios archivos<GUID2>(busque KustoLogs para ActivityID: , RootActivityId: )| Uno o más archivos no se han podido descargar. |Reintento |
|No se pudo analizar:<stream name>Stream with id ' ' tiene un formato Csv incorrecto, fallando según la directiva ValidationOptions |Archivo csv con formato incorrecto (por ejemplo, no el mismo número de columnas en cada línea). Solo se produce un error cuando la directiva de validación se establece en ValidationOptions. ValidateCsvInputConstantColumns |Compruebe los archivos csv. Este mensaje solo se aplica a los archivos csv/tsv |
|IngestClientAggregateException con el mensaje de error 'Falta parámetros obligatorios para la firma de acceso compartido válida' |La SAS que se utiliza es del servicio, y no de la cuenta de almacenamiento |Utilice la SAS de la cuenta de almacenamiento |

### <a name="ingestion-error-codes"></a>Códigos de error de ingestión

Para ayudar a controlar los errores de ingesta mediante programación, la información de error se enriquece con un código de error numérico (enumeración IngestionErrorCode).

|ErrorCode|Motivo|
|-----------|-------|
|Desconocido| Error desconocido|
|Stream_LowMemoryCondition| La operación se quedó sin memoria|
|Stream_WrongNumberOfFields| El documento CSV tiene un número incoherente de campos|
|Stream_InputStreamTooLarge| El documento presentado para la ingestión ha superado el tamaño permitido|
|Stream_NoDataToIngest| No se han encontrado flujos de datos para ingerir|
|Stream_DynamicPropertyBagTooLarge| Una de las columnas dinámicas de los datos ingeridos contiene demasiadas propiedades únicas|
|Download_SourceNotFound| No se pudo descargar el origen desde Azure Storage: origen no encontrado|
|Download_AccessConditionNotSatisfied| No se pudo descargar el origen desde Azure Storage: acceso denegado|
|Download_Forbidden| No se pudo descargar el origen desde Azure Storage: acceso prohibido|
|Download_AccountNotFound| No se pudo descargar el origen desde Azure Storage: no se ha encontrado la cuenta|
|Download_BadRequest| No se pudo descargar el origen desde Azure Storage: solicitud incorrecta|
|Download_NotTransient| No se pudo descargar el origen desde Azure Storage: no error transitorio|
|Download_UnknownError| No se pudo descargar el origen desde Azure Storage: error desconocido|
|UpdatePolicy_QuerySchemaDoesNotMatchTableSchema| No se pudo invocar la directiva de actualización. El esquema de consulta no coincide con el esquema de tabla|
|UpdatePolicy_FailedDescendantTransaction| No se pudo invocar la directiva de actualización. Directiva de actualización transaccional descendiente fallida|
|UpdatePolicy_IngestionError| No se pudo invocar la directiva de actualización. Error de ingestión|
|UpdatePolicy_UnknownError| No se pudo invocar la directiva de actualización. Error desconocido|
|BadRequest_MissingJsonMappingtFailure| El patrón Json no se ingsiió con el parámetro jsonMapping|
|BadRequest_InvalidOrEmptyBlob| Blob no es válido o archivo zip vacío|
|BadRequest_DatabaseNotExist| La base de datos no existe|
|BadRequest_TableNotExist| Tabla no existe|
|BadRequest_InvalidKustoIdentityToken| Token de identidad kusto no válido|
|BadRequest_UriMissingSas| Ruta de acceso de blobs sin SAS desde almacenamiento de blobs desconocido|
|BadRequest_FileTooLarge| Tratando de ingerir un archivo demasiado grande|
|BadRequest_NoValidResponseFromEngine| Ninguna respuesta válida del comando ingest|
|BadRequest_TableAccessDenied| Se deniega el acceso a la mesa|
|BadRequest_MessageExhausted| El mensaje se agota|
|General_BadRequest| Solicitud incorrecta general (puede indicar la ingestión a una base de datos/tabla no existente)|
|General_InternalServerError| Error interno del servidor|

## <a name="detailed-kustoingest-exceptions-reference"></a>Referencia detallada de excepciones Kusto.Ingest

### <a name="cloudqueuesnotfoundexception"></a>CloudQueuesNotFoundException
Se genera cuando no se devuelve ninguna cola desde el clúster de administración de datos

Clase base: [Excepción](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

Campos:

|Nombre|Tipo|Significado
|-----------|----|------------------------------|
|Error| `String`| El error que se produjo al intentar recuperar colas del DM
                            
Información adicional:

Sólo es relevante cuando se utiliza el cliente de [ingesta en cola de Kusto](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient).
Durante el proceso de ingesta se realizan varios intentos para recuperar las colas de Azure vinculadas a la DM. Cuando se produce un error en estos intentos, se produce la excepción que contiene el motivo del error en el campo 'Error' y posiblemente una excepción interna en el campo 'InnerException'.


### <a name="cloudblobcontainersnotfoundexception"></a>CloudBlobContainersNotFoundException
Se genera cuando no se devuelve ningún contenedor de blobs desde el clúster de administración de datos

Clase base: [Excepción](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

Campos:

|Nombre|Tipo|Significado       
|-----------|----|------------------------------|
|KustoEndpoint| `String`| El punto final del DM relevante
                            
Información adicional:

Sólo es relevante cuando se utiliza el cliente de [ingesta en cola de Kusto](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient).  
Al ingiriera orígenes que NO están ya en un contenedor de Azure, es decir, archivos, DataReader o Stream, los datos se cargan en un blob temporal para su ingesta. La excepción se produce cuando no se encuentran contenedores en los que cargar los datos.

### <a name="duplicateingestionpropertyexception"></a>DuplicateIngestionPropertyException
Se genera cuando una propiedad de ingesta se configura más de una vez

Clase base: [Excepción](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

Campos:

|Nombre|Tipo|Significado       
|-----------|----|------------------------------|
|PropertyName| `String`| El nombre de la propiedad duplicada
                            
### <a name="postmessagetoqueuefailedexception"></a>PostMessageToQueueFailedException
Error al publicar un mensaje en la cola

Clase base: [Excepción](https://msdn.microsoft.com/library/system.exception(v=vs.110).aspx)

Campos:

|Nombre|Tipo|Significado       
|-----------|----|------------------------------|
|QueueUri| `String`| El URI de la cola
|Error| `String`| El mensaje de error que se generó al intentar publicar en cola
                            
Información adicional:

Sólo es relevante cuando se utiliza el cliente de [ingesta en cola de Kusto](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient).  
El cliente de ingesta en cola ingesta datos cargando un mensaje en la cola de Azure correspondiente. En caso de error de publicación, se produce la excepción que contiene el URI de cola, el motivo del error en el campo 'Error' y posiblemente una excepción interna en el campo 'InnerException'.

### <a name="dataformatnotspecifiedexception"></a>DataFormatNotSpecifiedException
Se genera cuando se requiere formato de datos pero no se especifica en IngestionProperties

Clase base: IngestClientException

Información adicional:

Al ingerir desde una secuencia, se debe especificar un formato de datos en [IngestionProperties](kusto-ingest-client-reference.md#class-kustoingestionproperties) para ingerir correctamente los datos. Esta excepción se produce cuando no se especifica IngestionProperties.Format.

### <a name="invaliduriingestclientexception"></a>InvalidUriIngestClientException
Se genera cuando se envía un URI de blob no válido como origen de ingesta

Clase base: IngestClientException

### <a name="compressfileingestclientexception"></a>CompressFileIngestClientException
Se genera cuando el cliente de ingesta no pudo comprimir el archivo proporcionado para la ingesta

Clase base: IngestClientException

Información adicional:

Los archivos se comprimen antes de su ingestión. Esta excepción se produce cuando se produce un error al intentar comprimir el archivo.

### <a name="uploadfiletotempblobingestclientexception"></a>UploadFileToTempBlobIngestClientException
Se genera cuando el cliente de ingesta no pudo cargar el origen proporcionado para la ingesta en un blob temporal

Clase base: IngestClientException

### <a name="sizelimitexceededingestclientexception"></a>SizeLimitExceededIngestClientException
Se genera cuando una fuente de ingestión es demasiado grande

Clase base: IngestClientException

Campos:

|Nombre|Tipo|Significado       
|-----------|----|------------------------------|
|Tamaño| `long`| El tamaño de la fuente de ingestión
|MaxSize| `long`| El tamaño máximo permitido para la ingestión

Información adicional:

Si un origen de ingesta supera el tamaño máximo de 4 GB, se produce la excepción. La validación de tamaño se puede invalidar mediante la marca IgnoreSizeLimit en la [clase IngestionProperties](kusto-ingest-client-reference.md#class-kustoingestionproperties), pero no se recomienda [ingerir orígenes únicos de más](about-kusto-ingest.md#ingestion-best-practices)de 1 GB.

### <a name="uploadfiletotempblobingestclientexception"></a>UploadFileToTempBlobIngestClientException
Se genera cuando el cliente de ingesta no pudo cargar el archivo proporcionado para la ingesta en un blob temporal

Clase base: IngestClientException

### <a name="directingestclientexception"></a>DirectIngestClientException
Se genera cuando se produce un error general al realizar una ingestión directa

Clase base: IngestClientException

### <a name="queuedingestclientexception"></a>QueuedIngestClientException
Se genera cuando se produce un error al realizar una ingestión en cola

Clase base: IngestClientException

### <a name="ingestclientaggregateexception"></a>IngestClientAggregateException
Se genera cuando se produjeron uno o más errores durante una ingestión

Clase base: [AggregateException](https://msdn.microsoft.com/library/system.aggregateexception(v=vs.110).aspx)

Campos:

|Nombre|Tipo|Significado       
|-----------|----|------------------------------|
|IngestiónErrores| `IList<IngestClientException>`| Los errores que ocurrieron al intentar ingerir y las fuentes relacionadas con ellos
|IsGlobalError| `bool`| Indica si la excepción se produjo para todas las fuentes

## <a name="errors-in-native-code"></a>Errores en el código nativo
El motor Kusto está escrito en código nativo. Para obtener más información sobre los errores en el código nativo, consulte [Errores en código nativo](../../concepts/errorsinnativecode.md)
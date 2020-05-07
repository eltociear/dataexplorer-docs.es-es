---
title: 'Introducción a la ingesta de datos sin la biblioteca Kusto. ingesta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe cómo ingesta de datos sin la biblioteca Kusto. ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 02/19/2020
ms.openlocfilehash: 2ea7fd33a6e6ed8728fb12d53fbe76eadf8fd6b6
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82862078"
---
# <a name="howto-data-ingestion-without-kustoingest-library"></a>Cómo ingesta de datos sin la biblioteca Kusto. ingesta

## <a name="when-to-consider-not-using-kustoingest-library"></a>¿Cuándo considerar la posibilidad de no usar la biblioteca Kusto. ingesta?
Por lo general, se debe preferir el uso de la biblioteca Kusto. ingesta cuando se tenga en cuenta la ingesta de datos a Kusto.<BR>
Cuando esta no es una opción (normalmente debido a las restricciones del sistema operativo), con algún esfuerzo es posible lograr prácticamente la misma funcionalidad.<BR>
En este artículo se muestra cómo implementar la **ingesta en cola** en Kusto sin tener que depender del paquete Kusto. ingesta.

>**Nota:** El código siguiente se escribe en C#, haciendo uso de Azure Storage SDK, la biblioteca de autenticación de ADAL y el paquete NewtonSoft. JSON para simplificar el código de ejemplo.<BR>Si es necesario, el código correspondiente se puede reemplazar por las llamadas a la [API de REST de Azure Storage](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) apropiadas, el [paquete Adal de non-.net](https://docs.microsoft.com/azure/active-directory/develop/active-directory-authentication-libraries) y cualquier paquete de control de JSON disponible.

## <a name="overview"></a>Información general
En el ejemplo de código siguiente se muestra la ingesta de datos en cola (pasando a través de Kusto Administración de datos servicio) a Kusto sin el uso de la biblioteca Kusto. ingesta.<BR>
Esto puede ser útil si .NET completo es inaccesible o no está disponible debido al entorno u otras restricciones.<BR>

> En este artículo se trata el modo recomendado de ingesta para canalizaciones de nivel de producción, que también se conoce como **ingesta en cola** (en términos de la biblioteca Kusto. ingesta, la entidad correspondiente es la interfaz [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) ). En este modo, el código de cliente interactúa con el servicio Kusto mediante la publicación de mensajes de notificación de ingesta en una cola de Azure, referencia a la que se obtiene a partir del Administración de datos de Kusto (también conocido como Ingesta). La interacción con el servicio Administración de datos debe autenticarse con **AAD**.

El esquema de este flujo se describe en el ejemplo de código siguiente y se compone de lo siguiente:
1. Crear un cliente de Azure Storage y cargar los datos en un BLOB
1. Obtención del token de autenticación para obtener acceso al servicio de ingesta de Kusto
2. Consulte el servicio de ingesta de Kusto para obtener:
    * Recursos de ingesta (colas y contenedores de blobs)
    * Kusto token de identidad que se agregará a cada mensaje de ingesta
3. Carga de datos en un BLOB de uno de los contenedores de blobs obtenidos de Kusto en (2)
4. Cree un mensaje de ingesta que identifique la base de datos y la tabla de destino y apunte al BLOB de (3)
5. Publicar el mensaje de ingesta creado en (4) en una de las colas de ingesta obtenidas de Kusto en (2)
6. Recupera los errores detectados por el servicio durante la ingesta

En las secciones siguientes se explica cada paso con mayor detalle.

```csharp
// A container class for ingestion resources we are going to obtain from Kusto
internal class IngestionResourcesSnapshot
{
    public IList<string> IngestionQueues { get; set; } = new List<string>();
    public IList<string> TempStorageContainers { get; set; } = new List<string>();

    public string FailureNotificationsQueue { get; set; } = string.Empty;
    public string SuccessNotificationsQueue { get; set; } = string.Empty;
}

public static void IngestSingleFile(string file, string db, string table, string ingestionMappingRef)
{
    // Your Kusto ingestion service URI, typically ingest-<your cluster name>.kusto.windows.net
    string DmServiceBaseUri = @"https://ingest-{serviceNameAndRegion}.kusto.windows.net";

    // 1. Authenticate the interactive user (or application) to access Kusto ingestion service
    string bearerToken = AuthenticateInteractiveUser(DmServiceBaseUri);

    // 2a. Retrieve ingestion resources
    IngestionResourcesSnapshot ingestionResources = RetrieveIngestionResources(DmServiceBaseUri, bearerToken);

    // 2b. Retrieve Kusto identity token
    string identityToken = RetrieveKustoIdentityToken(DmServiceBaseUri, bearerToken);

    // 3. Upload file to one of the blob containers we got from Kusto.
    // This example uses the first one, but when working with multiple blobs,
    // one should round-robin the containers in order to prevent throttling
    long blobSizeBytes = 0;
    string blobName = $"TestData{DateTime.UtcNow.ToString("yyyy-MM-dd_HH-mm-ss.FFF")}";
    string blobUriWithSas = UploadFileToBlobContainer(file, ingestionResources.TempStorageContainers.First(),
                                                            "temp001", blobName, out blobSizeBytes);

    // 4. Compose ingestion command
    string ingestionMessage = PrepareIngestionMessage(db, table, blobUriWithSas, blobSizeBytes, ingestionMappingRef, identityToken);

    // 5. Post ingestion command to one of the ingestion queues we got from Kusto.
    // This example uses the first one, but when working with multiple blobs,
    // one should round-robin the queues in order to prevent throttling
    PostMessageToQueue(ingestionResources.IngestionQueues.First(), ingestionMessage);

    Thread.Sleep(20000);

    // 6a. Read success notifications
    var successes = PopTopMessagesFromQueue(ingestionResources.SuccessNotificationsQueue, 32);
    foreach (var sm in successes)
    {
        Console.WriteLine($"Ingestion completed: {sm}");
    }

    // 6b. Read failure notifications
    var errors = PopTopMessagesFromQueue(ingestionResources.FailureNotificationsQueue, 32);
    foreach (var em in errors)
    {
        Console.WriteLine($"Ingestion error: {em}");
    }
}
```

## <a name="1-obtain-authentication-evidence-from-aad"></a>1. obtener evidencia de autenticación de AAD
Aquí usamos ADAL para obtener un token de AAD para tener acceso al servicio Administración de datos de Kusto con el fin de solicitar sus colas de entrada.
ADAL está disponible en [plataformas que no son de Windows,](https://docs.microsoft.com/azure/active-directory/develop/active-directory-authentication-libraries) si es necesario.
```csharp
// Authenticates the interactive user and retrieves AAD Access token for specified resource
internal static string AuthenticateInteractiveUser(string resource)
{
    // Create Auth Context for MSFT AAD:
    AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/{AAD Tenant ID or name}");

    // Acquire user token for the interactive user for Kusto:
    AuthenticationResult result =
        authContext.AcquireTokenAsync(resource, "<your client app ID>", new Uri(@"<your client app URI>"),
                                        new PlatformParameters(PromptBehavior.Auto), UserIdentifier.AnyUser, "prompt=select_account").Result;
    return result.AccessToken;
}
```

## <a name="2-retrieve-kusto-ingestion-resources"></a>2. recuperar recursos de ingesta de Kusto
Aquí es donde las cosas son interesantes. Aquí construimos manualmente una solicitud HTTP POST a Kusto Administración de datos servicio, solicitando que se devuelvan los recursos de ingesta.
Entre ellas se incluyen las colas en las que el servicio DM está escuchando, así como los contenedores de blobs para la carga de datos.
El servicio de Administración de datos procesará todos los mensajes que contengan solicitudes de ingesta que lleguen a una de esas colas.
```csharp
// Retrieve ingestion resources (queues and blob containers) with SAS from specified Kusto Ingestion service using supplied Access token
internal static IngestionResourcesSnapshot RetrieveIngestionResources(string ingestClusterBaseUri, string accessToken)
{
    string ingestClusterUri = $"{ingestClusterBaseUri}/v1/rest/mgmt";
    string requestBody = $"{{ \"csl\": \".get ingestion resources\" }}";

    IngestionResourcesSnapshot ingestionResources = new IngestionResourcesSnapshot();

    using (WebResponse response = SendPostRequest(ingestClusterUri, accessToken, requestBody))
    using (StreamReader sr = new StreamReader(response.GetResponseStream()))
    using (JsonTextReader jtr = new JsonTextReader(sr))
    {
        JObject responseJson = JObject.Load(jtr);
        IEnumerable<JToken> tokens;

        // Input queues
        tokens = responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'SecuredReadyForAggregationQueue')]");
        foreach (var token in tokens)
        {
            ingestionResources.IngestionQueues.Add((string) token[1]);
        }

        // Temp storage containers
        tokens = responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'TempStorage')]");
        foreach (var token in tokens)
        {
            ingestionResources.TempStorageContainers.Add((string)token[1]);
        }

        // Failure notifications queue
        var singleToken =
            responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'FailedIngestionsQueue')].[1]").FirstOrDefault();
        ingestionResources.FailureNotificationsQueue = (string)singleToken;

        // Success notifications queue
        singleToken =
            responseJson.SelectTokens("Tables[0].Rows[?(@.[0] == 'SuccessfulIngestionsQueue')].[1]").FirstOrDefault();
        ingestionResources.SuccessNotificationsQueue = (string)singleToken;
    }

    return ingestionResources;
}

// Executes a POST request on provided URI using supplied Access token and request body
internal static WebResponse SendPostRequest(string uriString, string authToken, string body)
{
    WebRequest request = WebRequest.Create(uriString);

    request.Method = "POST";
    request.ContentType = "application/json";
    request.ContentLength = body.Length;
    request.Headers.Set(HttpRequestHeader.Authorization, $"Bearer {authToken}");

    Stream bodyStream = request.GetRequestStream();
    using (StreamWriter sw = new StreamWriter(bodyStream))
    {
        sw.Write(body);
        sw.Flush();
    }

    bodyStream.Close();
    return request.GetResponse();
}
```

## <a name="obtaining-kusto-identity-token"></a>Obtención del token de identidad de Kusto
Un paso importante en la autorización de la ingesta de datos es obtener un token de identidad y adjuntarlo a cada mensaje de introducción. Como los mensajes de introducción se entregan a Kusto a través de un canal no directo (cola de Azure), no hay ninguna manera de realizar la validación de la autorización en banda. El mecanismo de token de identidad permite esto mediante la emisión de una prueba de identidad con firma Kusto que puede ser validada por el servicio Kusto una vez que recibe el mensaje de ingesta.
```csharp
// Retrieves a Kusto identity token that will be added to every ingest message
internal static string RetrieveKustoIdentityToken(string ingestClusterBaseUri, string accessToken)
{
    string ingestClusterUri = $"{ingestClusterBaseUri}/v1/rest/mgmt";
    string requestBody = $"{{ \"csl\": \".get kusto identity token\" }}";
    string jsonPath = "Tables[0].Rows[*].[0]";

    using (WebResponse response = SendPostRequest(ingestClusterUri, accessToken, requestBody))
    using (StreamReader sr = new StreamReader(response.GetResponseStream()))
    using (JsonTextReader jtr = new JsonTextReader(sr))
    {
        JObject responseJson = JObject.Load(jtr);
        JToken identityToken = responseJson.SelectTokens(jsonPath).FirstOrDefault();

        return ((string)identityToken);
    }
}
```

## <a name="3-upload-data-to-azure-blob-container"></a>3. cargar datos en el contenedor de blobs de Azure
Este paso consiste en cargar un archivo local en un BLOB de Azure que se entregará más adelante para la ingesta. Este código emplea Azure Storage SDK, pero en los que esta dependencia no es posible, puede lograr la misma con la [API de REST de Azure BLOB Service](https://docs.microsoft.com/rest/api/storageservices/fileservices/blob-service-rest-api).
```csharp
// Uploads a single local file to an Azure Blob container, returns blob URI and original data size
internal static string UploadFileToBlobContainer(string filePath, string blobContainerUri, string containerName, string blobName, out long blobSize)
{
    var blobUri = new Uri(blobContainerUri);
    CloudBlobContainer blobContainer = new CloudBlobContainer(blobUri);
    CloudBlockBlob blockBlob = blobContainer.GetBlockBlobReference(blobName);

    using (Stream stream = File.OpenRead(filePath))
    {
        blockBlob.UploadFromStream(stream);
        blobSize = blockBlob.Properties.Length;
    }

    return string.Format("{0}{1}", blockBlob.Uri.AbsoluteUri, blobUri.Query);
}
```

## <a name="4-compose-kusto-ingestion-message"></a>4. Crear mensaje de ingesta de Kusto
Aquí usamos el paquete NewtonSoft. JSON para crear un mensaje de solicitud de ingesta válido que se publicará en la cola de Azure en la que está escuchando el servicio de Kusto Administración de datos adecuado.
* Este es el mínimo para el mensaje de ingesta.
* Tenga en cuenta que ese token de identidad es obligatorio y debe residir en el `AdditionalProperties` objeto JSON.
* Siempre que sea `CsvMapping` necesario, `JsonMapping` también se deben proporcionar las propiedades o.
* Consulte [el artículo sobre la creación previa de asignaciones de ingesta](../../management/create-ingestion-mapping-command.md) para obtener más detalles.
* En el [apéndice a](#appendix-a-ingestion-message-internal-structure) se proporciona una explicación sobre la estructura del mensaje de ingesta

```csharp
internal static string PrepareIngestionMessage(string db, string table, string dataUri, long blobSizeBytes, string mappingRef, string identityToken)
{
    var message = new JObject();

    message.Add("Id", Guid.NewGuid().ToString());
    message.Add("BlobPath", dataUri);
    message.Add("RawDataSize", blobSizeBytes);
    message.Add("DatabaseName", db);
    message.Add("TableName", table);
    message.Add("RetainBlobOnSuccess", true);   // Do not delete the blob on success
    message.Add("Format", "json");              // Data is in JSON format
    message.Add("FlushImmediately", true);      // Do not aggregate
    message.Add("ReportLevel", 2);              // Report failures and successes (might incur perf overhead)
    message.Add("ReportMethod", 0);             // Failures are reported to an Azure Queue

    message.Add("AdditionalProperties", new JObject(
                                            new JProperty("authorizationContext", identityToken),
                                            new JProperty("jsonMappingReference", mappingRef)));
    return message.ToString();
}
```

## <a name="5-post-kusto-ingestion-message-to-kusto-ingestion-queue"></a>5. publicar el mensaje de ingesta de Kusto en la cola de ingesta de Kusto
Y por último, el propio Deed-solo publica el mensaje que creamos en la cola que hemos elegido.<BR>
Nota: cuando se usa el cliente de almacenamiento de .net, codifica el mensaje en Base64 de forma predeterminada. Consulte [documentos de Storage](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.queue.cloudqueue.encodemessage).<BR>
Si no está usando ese cliente, asegúrese de codificar el contenido del mensaje correctamente.

```csharp
internal static void PostMessageToQueue(string queueUriWithSas, string message)
{
    CloudQueue queue = new CloudQueue(new Uri(queueUriWithSas));
    CloudQueueMessage queueMessage = new CloudQueueMessage(message);

    queue.AddMessage(queueMessage, null, null, null, null);
}
```

## <a name="6-pop-messages-from-an-azure-queue"></a>6. mensajes pop de una cola de Azure
Después de la ingesta, usamos este método para leer los mensajes de error de la cola adecuada que escribe el servicio Kusto Administración de datos.<BR>
En el [Apéndice B](#appendix-b-ingestion-failure-message-structure) se proporciona una explicación de la estructura de los mensajes de error

```csharp
internal static IEnumerable<string> PopTopMessagesFromQueue(string queueUriWithSas, int count)
{
    List<string> messages = Enumerable.Empty<string>().ToList();
    CloudQueue queue = new CloudQueue(new Uri(queueUriWithSas));
    var messagesFromQueue = queue.GetMessages(count);
    foreach (var m in messagesFromQueue)
    {
        messages.Add(m.AsString);
        queue.DeleteMessage(m);
    }

    return messages;
}
```

## <a name="appendix-a-ingestion-message-internal-structure"></a>Apéndice A: estructura interna del mensaje de ingesta
El mensaje que Kusto Administración de datos servicio espera leer de la cola de entrada de Azure es un documento JSON con el formato siguiente:

```json
{
    "Id" : "<Id>",
    "BlobPath" : "https://<AccountName>.blob.core.windows.net/<ContainerName>/<PathToBlob>?<SasToken>",
    "RawDataSize" : "<RawDataSizeInBytes>",
    "DatabaseName": "<DatabaseName>",
    "TableName" : "<TableName>",
    "RetainBlobOnSuccess" : "<RetainBlobOnSuccess>",
    "Format" : "<csv|tsv|...>",
    "FlushImmediately": "<true|false>",
    "ReportLevel" : <0-Failures, 1-None, 2-All>,
    "ReportMethod" : <0-Queue, 1-Table>,
    "AdditionalProperties" : { "<PropertyName>" : "<PropertyValue>" }
}
```


|Propiedad | Descripción |
|---------|-------------|
|Identificador |Identificador de mensaje (GUID) |
|BlobPath |URI de BLOB, incluida la clave de SAS que concede permisos de Kusto de lectura/escritura/eliminación (se requieren permisos de escritura/eliminación si Kusto es eliminar el BLOB una vez que ha terminado de ingerir los datos) |
|RawDataSize |Tamaño de los datos sin comprimir en bytes. Proporcionar este valor permite a Kusto optimizar la ingesta al agregar potencialmente varios blobs juntos. Esta propiedad es opcional, pero si no se proporciona, Kusto accederá al BLOB solo para recuperar el tamaño. |
|DatabaseName |Nombre de la base de datos de destino |
|TableName |Nombre de tabla de destino |
|RetainBlobOnSuccess |Si se establece `true`en, el BLOB no se eliminará cuando la ingesta se complete correctamente. De manera predeterminada, su valor es `false`. |
|Formato |Formato de datos sin comprimir |
|FlushImmediately |Si se establece `true`en, se omitirá cualquier agregación. De manera predeterminada, su valor es `false`. |
|ReportLevel |Nivel de informe de errores o correctos: 0-errores, 1-ninguno, 2-todos |
|ReportMethod |Mecanismo de informes: 0: cola, 1-tabla |
|AdditionalProperties |Cualquier propiedad adicional (etiquetas, etc.) |


## <a name="appendix-b-ingestion-failure-message-structure"></a>Apéndice B: estructura de los mensajes de error de ingesta
El siguiente mensaje de tabla que el servicio Kusto Administración de datos espera leer de la cola de entrada de Azure es un documento JSON con el formato siguiente:

|Propiedad | Descripción |
|---------|-------------
|OperationId |Identificador de operación (GUID) que se puede usar para realizar el seguimiento de la operación en el lado del servicio. |
|Base de datos |Nombre de la base de datos de destino |
|Tabla |Nombre de tabla de destino |
|Con errores |Marca de tiempo de error |
|IngestionSourceId |GUID que identifica el fragmento de datos que Kusto no pudo ingesta |
|IngestionSourcePath |Ruta de acceso (URI) al fragmento de datos que Kusto no pudo ingesta |
|Detalles |Mensaje de error |
|ErrorCode |Código de error Kusto (vea [aquí](kusto-ingest-client-errors.md#ingestion-error-codes)todos los códigos de error) |
|FailureStatus |Indica si el error es permanente o transitorio. |
|RootActivityId |Identificador de correlación (GUID) de Kusto que se puede usar para realizar el seguimiento de la operación en el lado del servicio. |
|OriginatesFromUpdatePolicy |Indica si el error se debe a una [Directiva de actualización transaccional](../../management/updatepolicy.md) erróneo |
|ShouldRetry | Indica si la ingesta puede realizarse correctamente si se vuelve a intentar como está |
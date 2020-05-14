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
ms.openlocfilehash: 80fe504311ee847afa7244e179974d80485efe46
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373553"
---
# <a name="ingestion-without-kustoingest-library"></a>Ingesta sin la biblioteca Kusto. ingesta

La biblioteca Kusto. ingesta es preferible para la ingesta de datos en Azure Explorador de datos. Sin embargo, todavía puede lograr prácticamente la misma funcionalidad, sin depender del paquete Kusto. ingesta.
En este artículo se muestra cómo usar la *ingesta en cola* en Azure explorador de datos para las canalizaciones de nivel de producción.

> [!NOTE]
> El código siguiente se escribe en C# y hace uso del SDK de Azure Storage, la biblioteca de autenticación de ADAL y el paquete NewtonSoft. JSON, para simplificar el código de ejemplo. Si es necesario, el código correspondiente se puede reemplazar por las llamadas a la [API de REST Azure Storage](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api) apropiadas, el [paquete Adal de non-.net](https://docs.microsoft.com/azure/active-directory/develop/active-directory-authentication-libraries)y cualquier paquete de control de JSON disponible.

En este artículo se trata el modo recomendado de ingesta. En el caso de la biblioteca Kusto. ingesta, su entidad correspondiente es la interfaz [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) . En este caso, el código de cliente interactúa con el servicio Azure Explorador de datos mediante la publicación de mensajes de notificación de ingesta en una cola de Azure. Las referencias a los mensajes se obtienen del servicio Kusto Administración de datos (también conocido como ingesta). La interacción con el servicio debe autenticarse con Azure Active Directory (Azure AD).

En el código siguiente se muestra cómo el servicio Kusto Administración de datos controla la ingesta de datos en cola sin usar la biblioteca Kusto. ingesta. Este ejemplo puede ser útil si .NET completo es inaccesible o no está disponible debido al entorno u otras restricciones.

El código incluye los pasos para crear un cliente de Azure Storage y cargar los datos en un BLOB.
Cada paso se describe con más detalle, después del código de ejemplo.

1. [Obtención de un token de autenticación para acceder al servicio de ingesta de Explorador de datos de Azure](#obtain-authentication-evidence-from-azure-ad)
1. Consulte el servicio Azure Explorador de datos ingesta para obtener:
    * [Recursos de ingesta (colas y contenedores de blobs)](#retrieve-azure-data-explorer-ingestion-resources)
    * [Un token de identidad de Kusto que se agregará a cada mensaje de ingesta.](#obtain-a-kusto-identity-token)
1. [Carga de datos en un BLOB de uno de los contenedores de blobs obtenidos de Kusto en (2)](#upload-data-to-the-azure-blob-container)
1. [Cree un mensaje de ingesta que identifique la base de datos y la tabla de destino y que apunte al BLOB de (3)](#compose-the-azure-data-explorer-ingestion-message)
1. [Publicar el mensaje de ingesta creado en (4) en una cola de ingesta obtenida de Azure Explorador de datos en (2)](#post-the-azure-data-explorer-ingestion-message-to-the-azure-data-explorer-ingestion-queue)**
1. [Recupera los errores encontrados por el servicio durante la ingesta](#check-for-error-messages-from-the-azure-queue)

```csharp
// A container class for ingestion resources we are going to obtain from Azure Data Explorer
internal class IngestionResourcesSnapshot
{
    public IList<string> IngestionQueues { get; set; } = new List<string>();
    public IList<string> TempStorageContainers { get; set; } = new List<string>();

    public string FailureNotificationsQueue { get; set; } = string.Empty;
    public string SuccessNotificationsQueue { get; set; } = string.Empty;
}

public static void IngestSingleFile(string file, string db, string table, string ingestionMappingRef)
{
    // Your Azure Data Explorer ingestion service URI, typically ingest-<your cluster name>.kusto.windows.net
    string DmServiceBaseUri = @"https://ingest-{serviceNameAndRegion}.kusto.windows.net";

    // 1. Authenticate the interactive user (or application) to access Kusto ingestion service
    string bearerToken = AuthenticateInteractiveUser(DmServiceBaseUri);

    // 2a. Retrieve ingestion resources
    IngestionResourcesSnapshot ingestionResources = RetrieveIngestionResources(DmServiceBaseUri, bearerToken);

    // 2b. Retrieve Kusto identity token
    string identityToken = RetrieveKustoIdentityToken(DmServiceBaseUri, bearerToken);

    // 3. Upload file to one of the blob containers we got from Azure Data Explorer.
    // This example uses the first one, but when working with multiple blobs,
    // one should round-robin the containers in order to prevent throttling
    long blobSizeBytes = 0;
    string blobName = $"TestData{DateTime.UtcNow.ToString("yyyy-MM-dd_HH-mm-ss.FFF")}";
    string blobUriWithSas = UploadFileToBlobContainer(file, ingestionResources.TempStorageContainers.First(),
                                                            "temp001", blobName, out blobSizeBytes);

    // 4. Compose ingestion command
    string ingestionMessage = PrepareIngestionMessage(db, table, blobUriWithSas, blobSizeBytes, ingestionMappingRef, identityToken);

    // 5. Post ingestion command to one of the ingestion queues we got from Azure Data Explorer.
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

## <a name="using-queued-ingestion-to-azure-data-explorer-for-production-grade-pipelines"></a>Uso de la ingesta en cola a Azure Explorador de datos para canalizaciones de nivel de producción

### <a name="obtain-authentication-evidence-from-azure-ad"></a>Obtener evidencia de autenticación de Azure AD

Aquí usamos ADAL para obtener un token de Azure AD para tener acceso al servicio Administración de datos de Kusto y solicitar sus colas de entrada.
ADAL está disponible en [plataformas que no son de Windows,](https://docs.microsoft.com/azure/active-directory/develop/active-directory-authentication-libraries) si es necesario.

```csharp
// Authenticates the interactive user and retrieves Azure AD Access token for specified resource
internal static string AuthenticateInteractiveUser(string resource)
{
    // Create Auth Context for MSFT Azure AD:
    AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/{Azure AD Tenant ID or name}");

    // Acquire user token for the interactive user for Azure Data Explorer:
    AuthenticationResult result =
        authContext.AcquireTokenAsync(resource, "<your client app ID>", new Uri(@"<your client app URI>"),
                                        new PlatformParameters(PromptBehavior.Auto), UserIdentifier.AnyUser, "prompt=select_account").Result;
    return result.AccessToken;
}
```

### <a name="retrieve-azure-data-explorer-ingestion-resources"></a>Recuperación de recursos de ingesta de Azure Explorador de datos

Construya manualmente una solicitud HTTP POST en el servicio de Administración de datos, solicitando la devolución de los recursos de ingesta. Estos recursos incluyen colas en las que el servicio DM está escuchando y contenedores de blobs para la carga de datos.
El servicio de Administración de datos procesará todos los mensajes que contengan solicitudes de ingesta que lleguen a una de esas colas.

```csharp
// Retrieve ingestion resources (queues and blob containers) with SAS from specified Azure Data Explorer Ingestion service using supplied Access token
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

### <a name="obtain-a-kusto-identity-token"></a>Obtención de un token de identidad de Kusto

Los mensajes de introducción se entregan a Azure Explorador de datos a través de un canal no directo (cola de Azure), lo que hace imposible realizar la validación de autorización en banda para tener acceso al servicio de ingesta de Explorador de datos de Azure. La solución consiste en adjuntar un token de identidad a cada mensaje de introducción. El token permite la validación de la autorización en banda. El servicio Azure Explorador de datos puede validar este token firmado cuando recibe el mensaje de ingesta.

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

### <a name="upload-data-to-the-azure-blob-container"></a>Carga de datos en el contenedor de blobs de Azure

Este paso consiste en cargar un archivo local en un BLOB de Azure que se entregará para la ingesta. Este código usa el SDK de Azure Storage. Si no es posible la dependencia, puede lograrse con la [API de REST de Azure BLOB Service](https://docs.microsoft.com/rest/api/storageservices/fileservices/blob-service-rest-api).

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

### <a name="compose-the-azure-data-explorer-ingestion-message"></a>Cree el mensaje de ingesta de Azure Explorador de datos

El paquete NewtonSoft. JSON volverá a crear una solicitud de ingesta válida para identificar la base de datos y la tabla de destino, y que apunta al BLOB.
El mensaje se publicará en la cola de Azure en la que está escuchando el servicio Kusto Administración de datos pertinente.

Estos son algunos puntos a tener en cuenta.

* Esta solicitud es el requisito mínimo para el mensaje de ingesta.

> [!NOTE]
> El token de identidad es obligatorio y debe formar parte del objeto JSON **AdditionalProperties** .

* Siempre que sea necesario, también se deben proporcionar las propiedades CsvMapping o JsonMapping
* Para obtener más información, consulte el [artículo sobre la creación previa de asignaciones de ingesta](../../management/create-ingestion-mapping-command.md).
* La [estructura interna del mensaje de ingesta](#ingestion-message-internal-structure) proporciona una explicación de la estructura del mensaje de ingesta

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

### <a name="post-the-azure-data-explorer-ingestion-message-to-the-azure-data-explorer-ingestion-queue"></a>Publicar el mensaje de ingesta de Azure Explorador de datos en la cola de ingesta de Azure Explorador de datos

Por último, publique el mensaje que ha creado en la cola de ingesta seleccionada que obtuvo de Azure Explorador de datos.

> [!NOTE]
> El cliente de almacenamiento de .net, cuando se usa, codifica el mensaje en Base64 de forma predeterminada. Para obtener más información, consulte [almacenamiento de documentos](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.queue.cloudqueue.encodemessage?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Queue_CloudQueue_EncodeMessage). Si no está usando ese cliente, asegúrese de codificar correctamente el contenido del mensaje.

```csharp
internal static void PostMessageToQueue(string queueUriWithSas, string message)
{
    CloudQueue queue = new CloudQueue(new Uri(queueUriWithSas));
    CloudQueueMessage queueMessage = new CloudQueueMessage(message);

    queue.AddMessage(queueMessage, null, null, null, null);
}
```

### <a name="check-for-error-messages-from-the-azure-queue"></a>Comprobar los mensajes de error de la cola de Azure

Después de la ingesta, se comprueban los mensajes de error de la cola correspondiente en la que escribe el Administración de datos. Para obtener más información sobre la estructura de mensajes de error, consulte [ingesta de mensajes de error](#ingestion-failure-message-structure). 

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

## <a name="ingestion-messages---json-document-formats"></a>Mensajes de ingesta: formatos de documento JSON

### <a name="ingestion-message-internal-structure"></a>Estructura interna del mensaje de ingesta

El mensaje que el servicio Administración de datos Kusto espera leer de la cola de entrada de Azure es un documento JSON con el formato siguiente.

```JSON
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
|BlobPath |Ruta de acceso (URI) al BLOB, incluida la clave de SAS que concede permisos de Explorador de datos de Azure para lectura, escritura y eliminación. Los permisos son necesarios para que Azure Explorador de datos pueda eliminar el BLOB una vez que haya completado la ingesta de los datos.|
|RawDataSize |Tamaño de los datos sin comprimir en bytes. Proporcionar este valor permite a Azure Explorador de datos optimizar la ingesta al agregar potencialmente varios BLOBs. Esta propiedad es opcional, pero si no se especifica, Azure Explorador de datos tendrá acceso al BLOB solo para recuperar el tamaño. |
|DatabaseName |Nombre de la base de datos de destino |
|TableName |Nombre de tabla de destino |
|RetainBlobOnSuccess |Si se establece en `true` , el BLOB no se eliminará cuando la ingesta se haya completado correctamente. Valor predeterminado: `false` |
|Formato |Formato de datos sin comprimir |
|FlushImmediately |Si se establece en `true` , se omitirá cualquier agregación. Valor predeterminado: `false` |
|ReportLevel |Nivel de informe de errores o correctos: 0-errores, 1-ninguno, 2-todos |
|ReportMethod |Mecanismo de informes: 0: cola, 1-tabla |
|AdditionalProperties |Propiedades adicionales como etiquetas |

### <a name="ingestion-failure-message-structure"></a>Estructura de mensaje de error de ingesta

El mensaje que el Administración de datos espera leer de la cola de entrada de Azure es un documento JSON con el formato siguiente.

|Propiedad | Descripción |
|---------|-------------
|OperationId |Identificador de operación (GUID) que se puede usar para realizar el seguimiento de la operación en el lado del servicio. |
|Base de datos |Nombre de la base de datos de destino |
|Tabla |Nombre de tabla de destino |
|Con errores |Marca de tiempo de error |
|IngestionSourceId |GUID que identifica el fragmento de datos que Azure Explorador de datos no pudo ingesta |
|IngestionSourcePath |Ruta de acceso (URI) al fragmento de datos que Azure Explorador de datos no pudo ingesta |
|Detalles |Mensaje de error |
|ErrorCode |Código de error de Azure Explorador de datos (vea [aquí](kusto-ingest-client-errors.md#ingestion-error-codes)todos los códigos de error) |
|FailureStatus |Indica si el error es permanente o transitorio. |
|RootActivityId |Identificador de correlación (GUID) de Explorador de datos de Azure que se puede usar para realizar el seguimiento de la operación en el lado del servicio. |
|OriginatesFromUpdatePolicy |Indica si el error se produjo por una [Directiva de actualización transaccional](../../management/updatepolicy.md) errónea |
|ShouldRetry | Indica si la ingesta puede realizarse correctamente si se vuelve a intentar como está |

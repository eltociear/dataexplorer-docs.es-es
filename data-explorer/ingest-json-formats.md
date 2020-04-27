---
title: Ingesta de datos con formato JSON en el Azure Data Explorer
description: Aprenda a ingerir datos con formato JSON en Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: kerend
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/27/2020
ms.openlocfilehash: 36c7616ee3d79cebaea96ca67787466cbd401cab
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663179"
---
# <a name="ingest-json-formatted-sample-data-into-azure-data-explorer"></a>Ingesta de datos de ejemplo con formato JSON en Azure Data Explorer

En este artículo, se muestra cómo ingerir datos con formato JSON en una base de datos de Azure Data Explorer. Comenzará con ejemplos sencillos de JSON asignado y sin formato, continuará con JSON multilínea y, a continuación, abordará esquemas JSON más complejos que contengan matrices y diccionarios.  En los ejemplos se detalla el proceso de ingesta de datos con formato JSON mediante el lenguaje de consulta de Kusto (KQL), C# o Python. Los comandos de control `ingest` del lenguaje de consulta Kusto se ejecutan directamente en el punto de conexión del motor. En escenarios de producción, la ingesta se ejecuta en el servicio Administración de datos mediante bibliotecas de clientes o conexiones de datos. Consulte [Ingesta de datos mediante la biblioteca de Python de Azure Data Explorer](/azure/data-explorer/python-ingest-data) e [Ingesta de datos mediante el SDK de .NET Standard de Azure Data Explorer](/azure/data-explorer/net-standard-ingest-data) para ver un tutorial sobre la ingesta de datos con estas bibliotecas de clientes.

## <a name="prerequisites"></a>Prerrequisitos

[Base de datos y clúster de prueba](create-cluster-database-portal.md)

## <a name="the-json-format"></a>El formato JSON

Azure Data Explorer admite dos formatos de archivo JSON:
* `json`: JSON separado por líneas. Cada línea de los datos de entrada tiene exactamente un registro JSON.
* `multijson`: JSON multilínea. El analizador omite los separadores de línea y lee un registro desde la posición anterior hasta el final de un JSON válido.

### <a name="ingest-and-map-json-formatted-data"></a>Ingesta y asignación de datos con formato JSON

La ingesta de datos con formato JSON requiere que especifique el *formato* mediante la [propiedad de ingesta](ingestion-properties.md). La ingesta de datos JSON requiere la [asignación](kusto/management/mappings.md), que asigna una entrada de origen JSON a su columna de destino. Al ingerir datos, use la propiedad de ingesta `jsonMappingReference` predefinida o especifique la propiedad de ingesta `jsonMapping`. En este artículo se usará la propiedad de ingesta `jsonMappingReference`, que está predefinida en la tabla que se utiliza para la ingesta. En los ejemplos siguientes, comenzaremos mediante la ingesta de registros JSON como datos sin procesar en una tabla de una sola columna. A continuación, usaremos la asignación para ingerir cada propiedad en su columna asignada. 

### <a name="simple-json-example"></a>Ejemplo de JSON simple

El siguiente ejemplo es un JSON simple, con una estructura plana. Los datos tienen información sobre la temperatura y la humedad, recopilados por varios dispositivos. Cada registro se marca con un identificador y una marca de tiempo.

```json
{
    "timestamp": "2019-05-02 15:23:50.0369439",
    "deviceId": "2945c8aa-f13e-4c48-4473-b81440bb5ca2",
    "messageId": "7f316225-839a-4593-92b5-1812949279b3",
    "temperature": 31.0301639051317,
    "humidity": 62.0791099602725
}
```

## <a name="ingest-raw-json-records"></a>Ingesta de registros JSON sin procesar 

En este ejemplo, se ingieren registros JSON como datos sin procesar en una tabla de una sola columna. La manipulación de datos, mediante consultas, y la directiva de actualización se realizan una vez que se ingieren los datos.

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

Use el lenguaje de consulta de Kusto para ingerir datos en formato JSON sin procesar.

1. Inicie sesión en [https://dataexplorer.azure.com](https://dataexplorer.azure.com).

1. Seleccione **Agregar clúster**.

1. En el cuadro de diálogo **Agregar clúster**, escriba la URL del clúster con el formato `https://<ClusterName>.<Region>.kusto.windows.net/` y después haga clic en **Agregar**.

1. Pegue el siguiente comando y seleccione **Ejecutar** para crear la tabla.

    ```kusto
    .create table RawEvents (Event: dynamic)
    ```

    Esta consulta crea una tabla con una sola columna `Event` de un [tipo de datos](kusto/query/scalar-data-types/dynamic.md) dinámico.

1. Cree la asignación de JSON.

    ```kusto
    .create table RawEvents ingestion json mapping 'RawEventMapping' '[{"column":"Event","path":"$"}]'
    ```

    Este comando crea una asignación y asigna la ruta de acceso raíz de JSON `$` a la columna `Event`.

1. Ingiera los datos en la tabla `RawEvents`.

    ```kusto
    .ingest into table RawEvents h'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=json, jsonMappingReference=RawEventMapping)
    ```

# <a name="c"></a>[C#](#tab/c-sharp)

Use C# para ingerir datos en formato JSON sin procesar.

1. Cree la tabla `RawEvents`.

    ```csharp
    var kustoUri = "https://<ClusterName>.<Region>.kusto.windows.net:443/";
    var kustoConnectionStringBuilder =
        new KustoConnectionStringBuilder(ingestUri)
        {
            FederatedSecurity = true,
            InitialCatalog = database,
            UserID = user,
            Password = password,
            Authority = tenantId
        };
    var kustoClient = KustoClientFactory.CreateCslAdminProvider(kustoConnectionStringBuilder);

    var table = "RawEvents";
    var command =
        CslCommandGenerator.GenerateTableCreateCommand(
            table,
            new[]
            {
                Tuple.Create("Events", "System.Object"),
            });

    kustoClient.ExecuteControlCommand(command);
    ```

1. Cree la asignación de JSON.
    
    ```csharp
    var tableMapping = "RawEventMapping";
    var command =
        CslCommandGenerator.GenerateTableJsonMappingCreateCommand(
            tableName,
            tableMapping,
            new[]
            {
                new JsonColumnMapping {ColumnName = "Events", JsonPath = "$"},
            });

    kustoClient.ExecuteControlCommand(command);
    ```
    Este comando crea una asignación y asigna la ruta de acceso raíz de JSON `$` a la columna `Event`.

1. Ingiera los datos en la tabla `RawEvents`.

    ```csharp
    var ingestUri = "https://ingest-<ClusterName>.<Region>.kusto.windows.net:443/";
    var blobPath = "https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var ingestConnectionStringBuilder =
        new KustoConnectionStringBuilder(ingestUri)
        {
            FederatedSecurity = true,
            InitialCatalog = database,
            UserID = user,
            Password = password,
            Authority = tenantId
        };
    var ingestClient = KustoIngestFactory.CreateQueuedIngestClient(ingestConnectionStringBuilder);
    
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.json,
            IngestionMappingReference = tableMapping
        };

    ingestClient.IngestFromSingleBlob(blobPath, deleteSourceOnSuccess: false, ingestionProperties: properties);
    ```

> [!NOTE]
> Los datos se agregan según la [directiva de procesamiento por lotes](kusto/management/batchingpolicy.md), lo que produce una latencia de unos minutos.

# <a name="python"></a>[Python](#tab/python)

Use Python para ingerir datos en formato JSON sin procesar.

1. Cree la tabla `RawEvents`.

    ```python
    KUSTO_URI = "https://<ClusterName>.<Region>.kusto.windows.net:443/"
    KCSB_DATA = KustoConnectionStringBuilder.with_aad_device_authentication(KUSTO_URI, AAD_TENANT_ID)
    KUSTO_CLIENT = KustoClient(KCSB_DATA)
    TABLE = "RawEvents"

    CREATE_TABLE_COMMAND = ".create table " + TABLE + " (Events: dynamic)"
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_TABLE_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. Cree la asignación de JSON.

    ```python
    MAPPING = "RawEventMapping"
    CREATE_MAPPING_COMMAND = ".create table " + TABLE + " ingestion json mapping '" + MAPPING + """' '[{"column":"Event","path":"$"}]'"""
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_MAPPING_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. Ingiera los datos en la tabla `RawEvents`.

    ```python
    INGEST_URI = "https://ingest-<ClusterName>.<Region>.kusto.windows.net:443/"
    KCSB_INGEST = KustoConnectionStringBuilder.with_aad_device_authentication(INGEST_URI, AAD_TENANT_ID)
    INGESTION_CLIENT = KustoIngestClient(KCSB_INGEST)
    BLOB_PATH = 'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.json, mappingReference=MAPPING)
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

    > [!NOTE]
    > Los datos se agregan según la [directiva de procesamiento por lotes](kusto/management/batchingpolicy.md), lo que produce una latencia de unos minutos.

---

## <a name="ingest-mapped-json-records"></a>Ingesta de registros JSON asignados

En este ejemplo, se ingieren datos de registros JSON. Cada propiedad JSON se asigna a una sola columna de la tabla. 

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

1. Cree una tabla con un esquema similar a los datos de entrada JSON. Usaremos esta tabla en todos los ejemplos y comandos de ingesta siguientes. 

    ```kusto
    .create table Events (Time: datetime, Device: string, MessageId: string, Temperature: double, Humidity: double)
    ```

1. Cree la asignación de JSON.

    ```kusto
    .create table Events ingestion json mapping 'FlatEventMapping' '[{"column":"Time","path":"$.timestamp"},{"column":"Device","path":"$.deviceId"},{"column":"MessageId","path":"$.messageId"},{"column":"Temperature","path":"$.temperature"},{"column":"Humidity","path":"$.humidity"}]'
    ```

    En esta asignación, tal y como se define en el esquema de tabla, las entradas `timestamp` se ingerirán en la columna `Time` como tipos de datos `datetime`.

1. Ingiera los datos en la tabla `Events`.

    ```kusto
    .ingest into table Events h'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=json, jsonMappingReference=FlatEventMapping)
    ```

    El archivo "simple.JSON" tiene algunos registros JSON separados por líneas. El formato es `json`, y la asignación utilizada en el comando de ingesta es el elemento `FlatEventMapping` que creó.

# <a name="c"></a>[C#](#tab/c-sharp)

1. Cree una tabla con un esquema similar a los datos de entrada JSON. Usaremos esta tabla en todos los ejemplos y comandos de ingesta siguientes. 

    ```csharp
    var table = "Events";
    var command =
        CslCommandGenerator.GenerateTableCreateCommand(
            table,
            new[]
            {
                Tuple.Create("Time", "System.DateTime"),
                Tuple.Create("Device", "System.String"),
                Tuple.Create("MessageId", "System.String"),
                Tuple.Create("Temperature", "System.Double"),
                Tuple.Create("Humidity", "System.Double"),
            });

    kustoClient.ExecuteControlCommand(command);
    ```

1. Cree la asignación de JSON.

    ```csharp
    var tableMapping = "FlatEventMapping";
    var command =
        CslCommandGenerator.GenerateTableJsonMappingCreateCommand(
            tableName,
            tableMapping,
            new[]
            {
                        new JsonColumnMapping {ColumnName = "Time", JsonPath = "$.timestamp"},
                        new JsonColumnMapping {ColumnName = "Device", JsonPath = "$.deviceId"},
                        new JsonColumnMapping {ColumnName = "MessageId", JsonPath = "$.messageId"},
                        new JsonColumnMapping {ColumnName = "Temperature", JsonPath = "$.temperature"},
                        new JsonColumnMapping {ColumnName = "Humidity", JsonPath = "$.humidity"},
            });

    kustoClient.ExecuteControlCommand(command);
    ```

    En esta asignación, tal y como se define en el esquema de tabla, las entradas `timestamp` se ingerirán en la columna `Time` como tipos de datos `datetime`.    

1. Ingiera los datos en la tabla `Events`.

    ```csharp
    var blobPath = "https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.json,
            IngestionMappingReference = tableMapping
        };

    ingestClient.IngestFromSingleBlob(blobPath, deleteSourceOnSuccess: false, ingestionProperties: properties);
    ```

    El archivo "simple.JSON" tiene algunos registros JSON separados por líneas. El formato es `json`, y la asignación utilizada en el comando de ingesta es el elemento `FlatEventMapping` que creó.

# <a name="python"></a>[Python](#tab/python)

1. Cree una tabla con un esquema similar a los datos de entrada JSON. Usaremos esta tabla en todos los ejemplos y comandos de ingesta siguientes. 

    ```python
    TABLE = "RawEvents"
    CREATE_TABLE_COMMAND = ".create table " + TABLE + " (Time: datetime, Device: string, MessageId: string, Temperature: double, Humidity: double)"
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_TABLE_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. Cree la asignación de JSON.

    ```python
    MAPPING = "FlatEventMapping"
    CREATE_MAPPING_COMMAND = ".create table Events ingestion json mapping '" + MAPPING + """' '[{"column":"Time","path":"$.timestamp"},{"column":"Device","path":"$.deviceId"},{"column":"MessageId","path":"$.messageId"},{"column":"Temperature","path":"$.temperature"},{"column":"Humidity","path":"$.humidity"}]'""" 
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_MAPPING_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. Ingiera los datos en la tabla `Events`.

    ```python
    BLOB_PATH = 'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/simple.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.json, mappingReference=MAPPING)
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

    El archivo "simple.JSON" tiene algunos registros JSON separados por líneas. El formato es `json`, y la asignación utilizada en el comando de ingesta es el elemento `FlatEventMapping` que creó.    
---

## <a name="ingest-multi-lined-json-records"></a>Ingesta de registros JSON multilínea

En este ejemplo, se ingieren registros JSON multilínea. Cada propiedad JSON se asigna a una sola columna de la tabla. El archivo "multilined.json" tiene algunos registros JSON con sangría. El formato `multijson` indica al motor que lea los registros por la estructura JSON.

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

Ingiera los datos en la tabla `Events`.

```kusto
.ingest into table Events h'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/multilined.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=multijson, jsonMappingReference=FlatEventMapping)
```

# <a name="c"></a>[C#](#tab/c-sharp)

Ingiera los datos en la tabla `Events`.

```csharp
var tableMapping = "FlatEventMapping";
var blobPath = "https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/multilined.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
var properties =
    new KustoQueuedIngestionProperties(database, table)
    {
        Format = DataSourceFormat.multijson,
        IngestionMappingReference = tableMapping
    };

ingestClient.IngestFromSingleBlob(blobPath, deleteSourceOnSuccess: false, ingestionProperties: properties);
```

# <a name="python"></a>[Python](#tab/python)

Ingiera los datos en la tabla `Events`.

```python
MAPPING = "FlatEventMapping"
BLOB_PATH = 'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/multilined.JSON?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.multijson, mappingReference=MAPPING)
BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
INGESTION_CLIENT.ingest_from_blob(
    BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
```

---

## <a name="ingest-json-records-containing-arrays"></a>Ingesta de registros JSON que contienen matrices

Los tipos de datos de matriz son una colección ordenada de valores. La ingesta de una matriz JSON se realiza mediante una [directiva de actualización](kusto/management/update-policy.md). El JSON se ingiere tal cual en una tabla intermedia. Una directiva de actualización ejecuta una función predefinida en la tabla `RawEvents`, de forma que se vuelven a ingerir los resultados en la tabla de destino. Se ingerirán datos con la siguiente estructura:

```json
{
    "records": 
    [
        {
            "timestamp": "2019-05-02 15:23:50.0000000",
            "deviceId": "ddbc1bf5-096f-42c0-a771-bc3dca77ac71",
            "messageId": "7f316225-839a-4593-92b5-1812949279b3",
            "temperature": 31.0301639051317,
            "humidity": 62.0791099602725
        },
        {
            "timestamp": "2019-05-02 15:23:51.0000000",
            "deviceId": "ddbc1bf5-096f-42c0-a771-bc3dca77ac71",
            "messageId": "57de2821-7581-40e4-861e-ea3bde102364",
            "temperature": 33.7529423105311,
            "humidity": 75.4787976739364
        }
    ]
}
```

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

1. Cree una función `update policy` que expanda la recopilación de datos de `records` para que cada valor de la recopilación reciba una fila independiente, mediante el operador `mv-expand`. Se usará la tabla `RawEvents` como tabla de origen y `Events` como tabla de destino.

    ```kusto
    .create function EventRecordsExpand() {
        RawEvents
        | mv-expand records = Event
        | project
            Time = todatetime(records["timestamp"]),
            Device = tostring(records["deviceId"]),
            MessageId = tostring(records["messageId"]),
            Temperature = todouble(records["temperature"]),
            Humidity = todouble(records["humidity"])
    }
    ```

1. El esquema que recibe la función debe coincidir con el esquema de la tabla de destino. Utilice el operador `getschema` para revisar el esquema.

    ```kusto
    EventRecordsExpand() | getschema
    ```

1. Agregue la directiva de actualización a la tabla de destino. Esta directiva ejecuta automáticamente la consulta en todos los datos recién ingeridos en la tabla intermedia `RawEvents` e ingiere los resultados en la tabla `Events`. Defina una directiva de retención cero para evitar que se conserve la tabla intermedia.

    ```kusto
    .alter table Events policy update @'[{"Source": "RawEvents", "Query": "EventRecordsExpand()", "IsEnabled": "True"}]'
    ```

1. Ingiera los datos en la tabla `RawEvents`.

    ```kusto
    .ingest into table RawEvents h'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/array.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=multijson, jsonMappingReference=RawEventMapping)
    ```

1. Revise los datos de la tabla `Events`.

    ```kusto
    Events
    ```

# <a name="c"></a>[C#](#tab/c-sharp)

1. Cree una función de actualización que expanda la recopilación de `records` para que cada valor de la recopilación reciba una fila independiente, mediante el operador `mv-expand`. Se usará la tabla `RawEvents` como tabla de origen y `Events` como tabla de destino.   

    ```csharp
    var command =
        CslCommandGenerator.GenerateCreateFunctionCommand(
            "EventRecordsExpand",
            "UpdateFunctions",
            string.Empty,
            null,
            @"RawEvents
                | mv-expand records = Event
                | project
                    Time = todatetime(records['timestamp']),
                    Device = tostring(records['deviceId']),
                    MessageId = tostring(records['messageId']),
                    Temperature = todouble(records['temperature']),
                    Humidity = todouble(records['humidity'])",
            ifNotExists: false);

    kustoClient.ExecuteControlCommand(command);
    ```

    > [!NOTE]
    > El esquema que recibe la función debe coincidir con el esquema de la tabla de destino.

1. Agregue la directiva de actualización a la tabla de destino. Esta directiva ejecuta automáticamente la consulta en todos los datos recién ingeridos en la tabla intermedia `RawEvents` e ingerirá los resultados en la tabla `Events`. Defina una directiva de retención cero para evitar que se conserve la tabla intermedia.

    ```csharp
    var command =
        ".alter table Events policy update @'[{'Source': 'RawEvents', 'Query': 'EventRecordsExpand()', 'IsEnabled': 'True'}]";

    kustoClient.ExecuteControlCommand(command);
    ```

1. Ingiera los datos en la tabla `RawEvents`.

    ```csharp
    var table = "RawEvents";
    var tableMapping = "RawEventMapping";
    var blobPath = "https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/array.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.multijson,
            IngestionMappingReference = tableMapping
        };

    ingestClient.IngestFromSingleBlob(blobPath, deleteSourceOnSuccess: false, ingestionProperties: properties);
    ```
    
1. Revise los datos de la tabla `Events`.

# <a name="python"></a>[Python](#tab/python)

1. Cree una función de actualización que expanda la recopilación de `records` para que cada valor de la recopilación reciba una fila independiente, mediante el operador `mv-expand`. Se usará la tabla `RawEvents` como tabla de origen y `Events` como tabla de destino.   

    ```python
    CREATE_FUNCTION_COMMAND = 
        '''.create function EventRecordsExpand() { 
            RawEvents
            | mv-expand records = Event
            | project
                Time = todatetime(records["timestamp"]),
                Device = tostring(records["deviceId"]),
                MessageId = tostring(records["messageId"]),
                Temperature = todouble(records["temperature"]),
                Humidity = todouble(records["humidity"])
            }'''
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_FUNCTION_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

    > [!NOTE]
    > El esquema que recibe la función debe coincidir con el esquema de la tabla de destino.

1. Agregue la directiva de actualización a la tabla de destino. Esta directiva ejecuta automáticamente la consulta en todos los datos recién ingeridos en la tabla intermedia `RawEvents` e ingerirá los resultados en la tabla `Events`. Defina una directiva de retención cero para evitar que se conserve la tabla intermedia.

    ```python
    CREATE_UPDATE_POLICY_COMMAND = 
        """.alter table Events policy update @'[{'Source': 'RawEvents', 'Query': 'EventRecordsExpand()', 'IsEnabled': 'True'}]"""
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_UPDATE_POLICY_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. Ingiera los datos en la tabla `RawEvents`.

    ```python
    TABLE = "RawEvents"
    MAPPING = "RawEventMapping"
    BLOB_PATH = 'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/array.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.multijson, mappingReference=MAPPING)
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

1. Revise los datos de la tabla `Events`.

---    

## <a name="ingest-json-records-containing-dictionaries"></a>Ingesta de registros JSON que contienen diccionarios

El JSON estructurado en diccionarios contiene pares de clave-valor. Los registros JSON pasan por una asignación de ingesta mediante la expresión lógica de `JsonPath`. Se pueden ingerir datos con la siguiente estructura:

```json
{
    "event": 
    [
        {
            "Key": "timestamp",
            "Value": "2019-05-02 15:23:50.0000000"
        },
        {
            "Key": "deviceId",
            "Value": "ddbc1bf5-096f-42c0-a771-bc3dca77ac71"
        },
        {
            "Key": "messageId",
            "Value": "7f316225-839a-4593-92b5-1812949279b3"
        },
        {
            "Key": "temperature",
            "Value": 31.0301639051317
        },
        {
            "Key": "humidity",
            "Value": 62.0791099602725
        }
    ]
}
```

# <a name="kql"></a>[KQL](#tab/kusto-query-language)

1. Cree una asignación JSON.

    ```kusto
    .create table Events ingestion json mapping 'KeyValueEventMapping' '[{"column":"Time","path":"$.event[?(@.Key == 'timestamp')]"},{"column":"Device","path":"$.event[?(@.Key == 'deviceId')]"},{"column":"MessageId","path":"$.event[?(@.Key == 'messageId')]"},{"column":"Temperature","path":"$.event[?(@.Key == 'temperature')]"},{"column":"Humidity","path":"$.event[?(@.Key == 'humidity')]"}]'
    ```

1. Ingiera los datos en la tabla `Events`.

    ```kusto
    .ingest into table Events h'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/dictionary.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (format=multijson, jsonMappingReference=KeyValueEventMapping)
    ```

# <a name="c"></a>[C#](#tab/c-sharp)

1. Cree una asignación JSON.

    ```csharp
    var tableName = "Events";
    var tableMapping = "KeyValueEventMapping";
    var command =
        CslCommandGenerator.GenerateTableJsonMappingCreateCommand(
            tableName,
            tableMapping,
            new[]
            {
                        new JsonColumnMapping {ColumnName = "Time", JsonPath = "$.event[?(@.Key == 'timestamp')]"},
                        new JsonColumnMapping {ColumnName = "Device", JsonPath = "$.event[?(@.Key == 'deviceId')]"},
                        new JsonColumnMapping {ColumnName = "MessageId", JsonPath = "$.event[?(@.Key == 'messageId')]"},
                        new JsonColumnMapping {ColumnName = "Temperature", JsonPath = "$.event[?(@.Key == 'temperature')]"},
                        new JsonColumnMapping {ColumnName = "Humidity", JsonPath = "$.event[?(@.Key == 'humidity')]"},
            });

    kustoClient.ExecuteControlCommand(command);
    ```

1. Ingiera los datos en la tabla `Events`.

    ```csharp
    var blobPath = "https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/dictionary.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D";
    var properties =
        new KustoQueuedIngestionProperties(database, table)
        {
            Format = DataSourceFormat.multijson,
            IngestionMappingReference = tableMapping
        };

    ingestClient.IngestFromSingleBlob(blobPath, deleteSourceOnSuccess: false, ingestionProperties: properties);
    ```

# <a name="python"></a>[Python](#tab/python)

1. Cree una asignación JSON.

    ```python
    MAPPING = "KeyValueEventMapping"
    CREATE_MAPPING_COMMAND = ".create table Events ingestion json mapping '" + MAPPING + """' '[{"column":"Time","path":"$.event[?(@.Key == 'timestamp')]"},{"column":"Device","path":"$.event[?(@.Key == 'deviceId')]"},{"column":"MessageId","path":"$.event[?(@.Key == 'messageId')]"},{"column":"Temperature","path":"$.event[?(@.Key == 'temperature')]"},{"column":"Humidity","path":"$.event[?(@.Key == 'humidity')]"}]'""" 
    RESPONSE = KUSTO_CLIENT.execute_mgmt(DATABASE, CREATE_MAPPING_COMMAND)
    dataframe_from_result_table(RESPONSE.primary_results[0])
    ```

1. Ingiera los datos en la tabla `Events`.

     ```python
    MAPPING = "KeyValueEventMapping"
    BLOB_PATH = 'https://kustosamplefiles.blob.core.windows.net/jsonsamplefiles/dictionary.json?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D'
    INGESTION_PROPERTIES = IngestionProperties(database=DATABASE, table=TABLE, dataFormat=DataFormat.multijson, mappingReference=MAPPING)u
    BLOB_DESCRIPTOR = BlobDescriptor(BLOB_PATH, FILE_SIZE)
    INGESTION_CLIENT.ingest_from_blob(
        BLOB_DESCRIPTOR, ingestion_properties=INGESTION_PROPERTIES)
    ```

---    

## <a name="next-steps"></a>Pasos siguientes

* [Información general de ingesta de datos](ingest-data-overview.md)
* [Escritura de consultas](write-queries.md)

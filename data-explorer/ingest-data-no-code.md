---
title: 'Tutorial: Ingesta de datos de supervisión en Azure Data Explorer sin código'
description: En este tutorial, obtendrá información sobre cómo ingerir datos de supervisión en Azure Data Explorer sin una línea de código y cómo consultar datos.
author: orspod
ms.author: orspodek
ms.reviewer: kerend
ms.service: data-explorer
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: 0808d0dadd410ae6d220b03ef54191192b925a21
ms.sourcegitcommit: ee90472a4f9d751d4049744d30e5082029c1b8fa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/21/2020
ms.locfileid: "83722191"
---
# <a name="tutorial-ingest-and-query-monitoring-data-in-azure-data-explorer"></a>Tutorial: Ingesta y consulta de datos de supervisión en Azure Data Explorer 

En este tutorial se muestra cómo ingerir datos de registros de actividad y diagnóstico para un clúster de Azure Data Explorer sin escribir código. Con este sencillo método de ingesta, puede empezar a consultar datos de Azure Data Explorer rápidamente para el análisis de datos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cree la asignación de la ingesta y las tablas en una base de datos de Azure Data Explorer.
> * Aplicar formato a los datos ingeridos mediante una directiva de actualización
> * Crear un [centro de eventos](/azure/event-hubs/event-hubs-about) y conectarlo a Azure Data Explorer
> * Hacer streaming de datos a un centro de eventos a partir de los [registros y métricas de diagnóstico](/azure/azure-monitor/platform/diagnostic-settings) y los [registros de actividad](/azure/azure-monitor/platform/activity-logs-overview) de Azure Monitor.
> * Consultar los datos ingeridos mediante Azure Data Explorer

> [!NOTE]
> Crear todos los recursos en la misma ubicación o región de Azure 

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Un clúster y la base de datos de Azure Data Explorer](create-cluster-database-portal.md). En este tutorial, el nombre de la base de datos es *TestDatabase*.

## <a name="azure-monitor-data-provider-diagnostic-metrics-and-logs-and-activity-logs"></a>Proveedor de datos de Azure Monitor: registros y métricas de diagnóstico y registros de actividad

Vea y comprenda los datos que proporcionan los siguientes registros y métricas de diagnóstico y registros de actividad de Azure Monitor. Va a crear una canalización de ingesta en función de estos esquemas de datos. Tenga en cuenta que cada evento de un registro tiene una matriz de registros. Esta matriz de registros se dividirá más adelante en el tutorial.

### <a name="examples-of-diagnostic-metrics-and-logs-and-activity-logs"></a>Ejemplos de registros y métricas de diagnóstico y registros de actividad

Los registros y métricas de diagnóstico y los registros de actividad de Azure son emitidos por un servicio de Azure que proporciona datos sobre el funcionamiento de ese servicio. 

# <a name="diagnostic-metrics"></a>[Métricas de diagnóstico](#tab/diagnostic-metrics)
#### <a name="example"></a>Ejemplo

Las métricas de diagnóstico se agregan con un intervalo de agregación de 1 minuto. El siguiente es un ejemplo de un esquema de eventos y métricas de Azure Data Explorer sobre la duración de una consulta:

```json
{
    "records": [
    {
        "count": 14,
        "total": 0,
        "minimum": 0,
        "maximum": 0,
        "average": 0,
        "resourceId": "/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.KUSTO/CLUSTERS/<cluster-name>",
        "time": "2018-12-20T17:00:00.0000000Z",
        "metricName": "QueryDuration",
        "timeGrain": "PT1M"
    },
    {
        "count": 12,
        "total": 0,
        "minimum": 0,
        "maximum": 0,
        "average": 0,
        "resourceId": "/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.KUSTO/CLUSTERS/<cluster-name>",
        "time": "2018-12-21T17:00:00.0000000Z",
        "metricName": "QueryDuration",
        "timeGrain": "PT1M"
    }
    ]
}
```

# <a name="diagnostic-logs"></a>[Registros de diagnóstico](#tab/diagnostic-logs)
#### <a name="example"></a>Ejemplo

El siguiente es un ejemplo de un [registro de ingesta de diagnósticos](using-diagnostic-logs.md#diagnostic-logs-schema) de Azure Data Explorer:

```json
{
    "time": "2019-08-26T13:22:36.8804326Z",
    "resourceId": "/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.KUSTO/CLUSTERS/<cluster-name>",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "FailedIngestion",
    "resultType": "Failed",
    "correlationId": "d59882f1-ad64-4fc4-b2ef-d663b6cc1cc5",
    "properties": {
        "OperationId": "00000000-0000-0000-0000-000000000000",
        "Database": "Kusto",
        "Table": "Table_13_20_prod",
        "FailedOn": "2019-08-26T13:22:36.8804326Z",
        "IngestionSourceId": "d59882f1-ad64-4fc4-b2ef-d663b6cc1cc5",
        "Details":
        {
            "error": 
            {
                "code": "BadRequest_DatabaseNotExist",
                "message": "Request is invalid and cannot be executed.",
                "@type": "Kusto.Data.Exceptions.DatabaseNotFoundException",
                "@message": "Database 'Kusto' was not found.",
                "@context": 
                {
                    "timestamp": "2019-08-26T13:22:36.7179157Z",
                    "serviceAlias": "<cluster-name>",
                    "machineName": "KEngine000001",
                    "processName": "Kusto.WinSvc.Svc",
                    "processId": 5336,
                    "threadId": 6528,
                    "appDomainName": "Kusto.WinSvc.Svc.exe",
                    "clientRequestd": "DM.IngestionExecutor;a70ddfdc-b471-4fc7-beac-bb0f6e569fe8",
                    "activityId": "f13e7718-1153-4e65-bf82-8583d712976f",
                    "subActivityId": "2cdad9d0-737b-4c69-ac9a-22cf9af0c41b",
                    "activityType": "DN.AdminCommand.DataIngestPullCommand",
                    "parentActivityId": "2f65e533-a364-44dd-8d45-d97460fb5795",
                    "activityStack": "(Activity stack: CRID=DM.IngestionExecutor;a70ddfdc-b471-4fc7-beac-bb0f6e569fe8 ARID=f13e7718-1153-4e65-bf82-8583d712976f > DN.Admin.Client.ExecuteControlCommand/5b764b32-6017-44a2-89e7-860eda515d40 > P.WCF.Service.ExecuteControlCommandInternal..IAdminClientServiceCommunicationContract/c2ef9344-069d-44c4-88b1-a3570697ec77 > DN.FE.ExecuteControlCommand/2f65e533-a364-44dd-8d45-d97460fb5795 > DN.AdminCommand.DataIngestPullCommand/2cdad9d0-737b-4c69-ac9a-22cf9af0c41b)"
                },
                "@permanent": true
            }
        },
        "ErrorCode": "BadRequest_DatabaseNotExist",
        "FailureStatus": "Permanent",
        "RootActivityId": "00000000-0000-0000-0000-000000000000",
        "OriginatesFromUpdatePolicy": false,
        "ShouldRetry": false,
        "IngestionSourcePath": "https://c0skstrldkereneus01.blob.core.windows.net/aam-20190826-temp-e5c334ee145d4b43a3a2d3a96fbac1df/3216_test_3_columns_invalid_8f57f0d161ed4a8c903c6d1073005732_59951f9ca5d143b6bdefe52fa381a8ca.zip"
    }
}
```
# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)
#### <a name="example"></a>Ejemplo

Los registros de actividad de Azure son registros de nivel de suscripción que proporcionan información sobre las operaciones realizadas en los recursos de la suscripción. Este es un ejemplo de un evento del registro de actividad para la comprobación de acceso:

```json
{
    "records": [
    {
        "time": "2018-12-26T16:23:06.1090193Z",
        "resourceId": "/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.WEB/SITES/CLNB5F73B70-DCA2-47C2-BB24-77B1A2CAAB4D/PROVIDERS/MICROSOFT.AUTHORIZATION",
        "operationName": "MICROSOFT.AUTHORIZATION/CHECKACCESS/ACTION",
        "category": "Action",
        "resultType": "Start",
        "resultSignature": "Started.",
        "durationMs": 0,
        "callerIpAddress": "13.66.225.188",
        "correlationId": "0de9f4bc-4adc-4209-a774-1b4f4ae573ed",
        "identity": {
            "authorization": {
                ...
            },
            "claims": {
                ...
            }
        },
        "level": "Information",
        "location": "global",
        "properties": {
            ...
        }
    },
    {
        "time": "2018-12-26T16:23:06.3040244Z",
        "resourceId": "/SUBSCRIPTIONS/<subscriptionID>/RESOURCEGROUPS/<resource-group>/PROVIDERS/MICROSOFT.WEB/SITES/CLNB5F73B70-DCA2-47C2-BB24-77B1A2CAAB4D/PROVIDERS/MICROSOFT.AUTHORIZATION",
        "operationName": "MICROSOFT.AUTHORIZATION/CHECKACCESS/ACTION",
        "category": "Action",
        "resultType": "Success",
        "resultSignature": "Succeeded.OK",
        "durationMs": 194,
        "callerIpAddress": "13.66.225.188",
        "correlationId": "0de9f4bc-4adc-4209-a774-1b4f4ae573ed",
        "identity": {
            "authorization": {
                ...
            },
            "claims": {
                ...
            }
        },
        "level": "Information",
        "location": "global",
        "properties": {
            "statusCode": "OK",
            "serviceRequestId": "87acdebc-945f-4c0c-b931-03050e085626"
        }
    }]
}
```
---

## <a name="set-up-an-ingestion-pipeline-in-azure-data-explorer"></a>Configuración de una canalización de ingesta en Azure Data Explorer

La configuración de una canalización de Azure Data Explorer implica varios pasos, como la [creación de tablas y la ingesta de datos](ingest-sample-data.md#ingest-data). También puede manipular, asignar y actualizar los datos.

### <a name="connect-to-the-azure-data-explorer-web-ui"></a>Conexión a la interfaz de usuario web de Azure Data Explorer

En la base de datos *TestDatabase* de Azure Data Explorer, seleccione **Query** (Consultar) para abrir la interfaz de usuario web de Azure Data Explorer.

![Página de consulta](media/ingest-data-no-code/query-database.png)

### <a name="create-the-target-tables"></a>Creación de las tablas de destino

La estructura de los registros de Azure Monitor no es tabular. Podrá manipular los datos y expandir cada evento a uno o más registros. Los datos sin procesar se ingieren en una tabla intermedia denominada *ActivityLogsRawRecords* para registros de actividad y *DiagnosticRawRecords* para registros y métricas de diagnóstico. En ese momento, los datos se manipularán y se expandirán. Mediante una directiva de actualización, los datos expandidos se ingieren en la tabla *ActivityLogs* para los registros de actividad, *DiagnosticMetrics* para las métricas de diagnóstico y *DiagnosticLogs* para los registros de diagnóstico. Esto significa que deberá crear dos tablas independientes para la ingesta de los registros de actividad y dos tablas independientes para la ingesta de los registros y métricas de diagnóstico.

Use la interfaz de usuario web de Azure Data Explorer para crear las tablas de destino en la base de datos de Azure Data Explorer.

# <a name="diagnostic-metrics"></a>[Métricas de diagnóstico](#tab/diagnostic-metrics)
#### <a name="create-tables-for-the-diagnostic-metrics"></a>Creación de tablas para las métricas de diagnóstico

1. En la base de datos *TestDatabase*, cree una tabla llamada *DiagnosticMetrics* para almacenar los registros de métricas de diagnóstico. Use el siguiente comando de control `.create table`:

    ```kusto
    .create table DiagnosticMetrics (Timestamp:datetime, ResourceId:string, MetricName:string, Count:int, Total:double, Minimum:double, Maximum:double, Average:double, TimeGrain:string)
    ```

1. Seleccione **Ejecutar** para crear la tabla.

    ![Ejecutar consulta](media/ingest-data-no-code/run-query.png)

1. Cree la tabla de datos intermedia llamada *DiagnosticRawRecords* en la base de datos *TestDatabase* para la manipulación de los datos mediante la siguiente consulta. Seleccione **Ejecutar** para crear la tabla.

    ```kusto
    .create table DiagnosticRawRecords (Records:dynamic)
    ```

1. Establezca una [directiva de retención](kusto/management/retention-policy.md) cero para la tabla intermedia:

    ```kusto
    .alter-merge table DiagnosticRawRecords policy retention softdelete = 0d
    ```

# <a name="diagnostic-logs"></a>[Registros de diagnóstico](#tab/diagnostic-logs)
#### <a name="create-tables-for-the-diagnostic-logs"></a>Creación de tablas para los registros de diagnóstico 

1. En la base de datos *TestDatabase*, cree una tabla llamada *DiagnosticLogs* para almacenar los registros de diagnóstico. Use el siguiente comando de control `.create table`:

    ```kusto
    .create table DiagnosticLogs (Timestamp:datetime, ResourceId:string, OperationName:string, Result:string, OperationId:string, Database:string, Table:string, IngestionSourceId:string, IngestionSourcePath:string, RootActivityId:string, ErrorCode:string, FailureStatus:string, Details:string)
    ```

1. Seleccione **Ejecutar** para crear la tabla.

1. Cree la tabla de datos intermedia llamada *DiagnosticRawRecords* en la base de datos *TestDatabase* para la manipulación de los datos mediante la siguiente consulta. Seleccione **Ejecutar** para crear la tabla.

    ```kusto
    .create table DiagnosticRawRecords (Records:dynamic)
    ```

1. Establezca una [directiva de retención](kusto/management/retention-policy.md) cero para la tabla intermedia:

    ```kusto
    .alter-merge table DiagnosticRawRecords policy retention softdelete = 0d
    ```

# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)
#### <a name="create-tables-for-the-activity-logs"></a>Creación de tablas para los registros de actividad 

1. Cree una tabla llamada *ActivityLogs* en la base de datos *TestDatabase* para recibir registros de actividad. Para crear la tabla, ejecute la siguiente consulta de Azure Data Explorer:

    ```kusto
    .create table ActivityLogs (Timestamp:datetime, ResourceId:string, OperationName:string, Category:string, ResultType:string, ResultSignature:string, DurationMs:int, IdentityAuthorization:dynamic, IdentityClaims:dynamic, Location:string, Level:string)
    ```

1. Cree la tabla de datos intermedia llamada *ActivityLogsRawRecords* en la base de datos *TestDatabase* para la manipulación de los datos:

    ```kusto
    .create table ActivityLogsRawRecords (Records:dynamic)
    ```

1. Establezca una [directiva de retención](kusto/management/retention-policy.md) cero para la tabla intermedia:

    ```kusto
    .alter-merge table ActivityLogsRawRecords policy retention softdelete = 0d
    ```
---

### <a name="create-table-mappings"></a>Creación de asignaciones de tablas

 Como el formato de datos es `json`, se requiere la asignación de datos. La asignación `json` asigna cada ruta de acceso JSON a un nombre de columna de tabla.

# <a name="diagnostic-metrics--diagnostic-logs"></a>[Métricas de diagnóstico y registros de diagnóstico](#tab/diagnostic-metrics+diagnostic-logs) 
#### <a name="map-diagnostic-metrics-and-logs-to-the-table"></a>Asignación de métricas y registros de diagnóstico a la tabla

Para asignar las métricas de diagnóstico y los datos del registro a la tabla, use la siguiente consulta:

```kusto
.create table DiagnosticRawRecords ingestion json mapping 'DiagnosticRawRecordsMapping' '[{"column":"Records","Properties":{"path":"$.records"}}]'
```

# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)
#### <a name="map-activity-logs-to-the-table"></a>Asignación de registros de actividad a la tabla

Para asignar los datos de los registros de actividad a la tabla, use la siguiente consulta:

```kusto
.create table ActivityLogsRawRecords ingestion json mapping 'ActivityLogsRawRecordsMapping' '[{"column":"Records","Properties":{"path":"$.records"}}]'
```
---

### <a name="create-the-update-policy-for-metric-and-log-data"></a>Creación de la directiva de actualización para los datos de métricas y registros

# <a name="diagnostic-metrics"></a>[Métricas de diagnóstico](#tab/diagnostic-metrics)
#### <a name="create-data-update-policy-for-diagnostics-metrics"></a>Creación de una directiva de actualización de datos para métricas de diagnóstico

1. Cree una [función](kusto/management/functions.md) que expanda la colección de registros de métricas de diagnóstico para que cada valor de la colección reciba una fila independiente. Use el operador [`mv-expand`](kusto/query/mvexpandoperator.md):
     ```kusto
    .create function DiagnosticMetricsExpand() {
        DiagnosticRawRecords
        | mv-expand events = Records
        | where isnotempty(events.metricName)
        | project
            Timestamp = todatetime(events['time']),
            ResourceId = tostring(events.resourceId),
            MetricName = tostring(events.metricName),
            Count = toint(events['count']),
            Total = todouble(events.total),
            Minimum = todouble(events.minimum),
            Maximum = todouble(events.maximum),
            Average = todouble(events.average),
            TimeGrain = tostring(events.timeGrain)
    }
    ```

2. Agregue la [directiva de actualización](kusto/management/updatepolicy.md) a la tabla de destino. Esta directiva ejecuta automáticamente la consulta en cualquier dato recién ingerido en la tabla de datos intermedia *DiagnosticRawRecords* e ingiere sus resultados en la tabla *DiagnosticMetrics*:

    ```kusto
    .alter table DiagnosticMetrics policy update @'[{"Source": "DiagnosticRawRecords", "Query": "DiagnosticMetricsExpand()", "IsEnabled": "True", "IsTransactional": true}]'
    ```

# <a name="diagnostic-logs"></a>[Registros de diagnóstico](#tab/diagnostic-logs)
#### <a name="create-data-update-policy-for-diagnostics-logs"></a>Creación de una directiva de actualización de datos para registros de diagnóstico

1. Cree una [función](kusto/management/functions.md) que expanda la colección de registros de diagnóstico para que cada valor de la colección reciba una fila independiente. Va a habilitar los registros de ingesta en un clúster de Azure Data Explorer y usará un [esquema de registros de ingesta](using-diagnostic-logs.md#diagnostic-logs-schema). Creará una tabla para ingestas correctas y erróneas, aunque algunos de los campos estarán vacíos en el caso de las correctas (ErrorCode, por ejemplo). Use el operador [`mv-expand`](kusto/query/mvexpandoperator.md):

    ```kusto
    .create function DiagnosticLogsExpand() {
        DiagnosticRawRecords
        | mv-expand events = Records
        | where isnotempty(events.operationName)
        | project
            Timestamp = todatetime(events['time']),
            ResourceId = tostring(events.resourceId),
            OperationName = tostring(events.operationName),
            Result = tostring(events.resultType),
            OperationId = tostring(events.properties.OperationId),
            Database = tostring(events.properties.Database),
            Table = tostring(events.properties.Table),
            IngestionSourceId = tostring(events.properties.IngestionSourceId),
            IngestionSourcePath = tostring(events.properties.IngestionSourcePath),
            RootActivityId = tostring(events.properties.RootActivityId),
            ErrorCode = tostring(events.properties.ErrorCode),
            FailureStatus = tostring(events.properties.FailureStatus),
            Details = tostring(events.properties.Details)
    }
    ```

2. Agregue la [directiva de actualización](kusto/management/updatepolicy.md) a la tabla de destino. Esta directiva ejecuta automáticamente la consulta en cualquier dato recién ingerido en la tabla de datos intermedia *DiagnosticRawRecords* e ingiere sus resultados en la tabla *DiagnosticLogs*:

    ```kusto
    .alter table DiagnosticLogs policy update @'[{"Source": "DiagnosticRawRecords", "Query": "DiagnosticLogsExpand()", "IsEnabled": "True", "IsTransactional": true}]'
    ```

# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)
#### <a name="create-data-update-policy-for-activity-logs"></a>Creación de una directiva de actualización de datos de los registros de actividad

1. Cree una [función](kusto/management/functions.md) que expanda la colección de registros de actividad para que cada valor de la colección reciba una fila independiente. Use el operador [`mv-expand`](kusto/query/mvexpandoperator.md):

    ```kusto
    .create function ActivityLogRecordsExpand() {
        ActivityLogsRawRecords
        | mv-expand events = Records
        | project
            Timestamp = todatetime(events['time']),
            ResourceId = tostring(events.resourceId),
            OperationName = tostring(events.operationName),
            Category = tostring(events.category),
            ResultType = tostring(events.resultType),
            ResultSignature = tostring(events.resultSignature),
            DurationMs = toint(events.durationMs),
            IdentityAuthorization = events.identity.authorization,
            IdentityClaims = events.identity.claims,
            Location = tostring(events.location),
            Level = tostring(events.level)
    }
    ```

2. Agregue la [directiva de actualización](kusto/management/updatepolicy.md) a la tabla de destino. Esta directiva ejecuta automáticamente la consulta en cualquier dato recién ingerido en la tabla de datos intermedia *ActivityLogsRawRecords* e ingiere sus resultados en la tabla *ActivityLogs*:

    ```kusto
    .alter table ActivityLogs policy update @'[{"Source": "ActivityLogsRawRecords", "Query": "ActivityLogRecordsExpand()", "IsEnabled": "True", "IsTransactional": true}]'
    ```
---

## <a name="create-an-azure-event-hubs-namespace"></a>Creación de un espacio de nombres de Azure Event Hubs

La configuración de diagnóstico de Azure permite exportar métricas y registros a una cuenta de almacenamiento o a un centro de eventos. En este tutorial, enrutará las métricas y registros mediante un centro de eventos. En los pasos siguientes creará un espacio de nombres de Event Hubs y un centro de eventos para las métricas y registros de diagnóstico. Azure Monitor creará el centro de eventos *insights-operational-logs* para los registros de actividad.

1. Cree un centro de eventos mediante una plantilla de Azure Resource Manager en Azure Portal. Para seguir el resto de los pasos de este artículo, haga clic con el botón derecho en el botón **Implementar en Azure** y, luego, seleccione **Abrir en una nueva ventana**. El botón **Deploy to Azure** (Implementar en Azure) le lleva a Azure Portal.

    [![Botón Implementar en Azure](media/ingest-data-no-code/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-event-hubs-create-event-hub-and-consumer-group%2Fazuredeploy.json)

1. Cree un espacio de nombres de Event Hubs y un centro de eventos para los registros de diagnóstico.

    ![Creación de un centro de eventos](media/ingest-data-no-code/event-hub.png)

1. Rellene el formulario con la siguiente información. Para el caso de opciones de configuración que no se muestren en la siguiente tabla, use los valores predeterminados.

    **Configuración** | **Valor sugerido** | **Descripción**
    |---|---|---|
    | **Suscripción** | *Su suscripción* | Seleccione la suscripción de Azure que quiere usar para el centro de eventos.|
    | **Grupos de recursos** | *test-resource-group* | Cree un nuevo grupo de recursos. |
    | **Ubicación** | Seleccione la región que mejor se adapte a sus necesidades. | Cree el espacio de nombres de Event Hubs en la misma ubicación que otros recursos.
    | **Nombre del espacio de nombres** | *AzureMonitoringData* | Elija un nombre único que identifique el espacio de nombres.
    | **Nombre del centro de eventos** | *DiagnosticData* | El centro de eventos se encuentra bajo el espacio de nombres, que proporciona un contenedor de ámbito único. |
    | **Nombre del grupo de consumidores** | *adxpipeline* | Cree un nombre para el grupo de consumidores. Los grupos de consumidores permiten consumir varias aplicaciones y que cada una tenga una vista independiente del flujo de eventos. |
    | | |

## <a name="connect-azure-monitor-metrics-and-logs-to-your-event-hub"></a>Conexión de métricas y registros de Azure Monitor al centro de eventos

Ahora debe conectar los registros y métricas de diagnóstico y los registros de actividad al centro de eventos.

# <a name="diagnostic-metrics--diagnostic-logs"></a>[Métricas de diagnóstico y registros de diagnóstico](#tab/diagnostic-metrics+diagnostic-logs) 
### <a name="connect-diagnostic-metrics-and-logs-to-your-event-hub"></a>Conexión de métricas y registros de diagnóstico al centro de eventos

Seleccione un recurso desde el que se va a exportar las métricas. Hay varios tipos de recursos que admiten la exportación de datos de diagnóstico, incluido el espacio de nombres de Event Hubs, Azure Key Vault, Azure IoT Hub y los clústeres de Azure Data Explorer. En este tutorial, se utilizará un clúster de Azure Data Explorer como nuestro recurso, se revisarán las métricas de rendimiento de las consultas y los registros de resultados de la ingesta.

1. Seleccione el clúster de Kusto en Azure Portal.
1. Seleccione **Configuración de diagnóstico** y, luego, el vínculo **Activar diagnóstico**. 

    ![Configuración de diagnóstico](media/ingest-data-no-code/diagnostic-settings.png)

1. Se abre el panel **Configuración de diagnóstico**. Siga estos pasos.
   1. Asigne a los datos del registro de diagnóstico el nombre *ADXExportedData*.
   1. En **REGISTRO**, active las casillas **SucceededIngestion** y **FailedIngestion**.
   1. En **MÉTRICA**, seleccione la casilla **Query performance**.
   1. Active la casilla **Transmitir a un centro de eventos**.
   1. Seleccione **Configurar**.

      ![Panel Configuración de diagnóstico](media/ingest-data-no-code/diagnostic-settings-window.png)

1. En el panel **Seleccionar centro de eventos**, configure cómo exportar datos desde los registros de diagnóstico hasta el centro de eventos que ha creado:
    1. En la lista **Seleccionar el espacio de nombres del Centro de eventos**, seleccione *AzureMonitoringData*.
    1. En la lista **Seleccionar nombre de centro de eventos**, seleccione *DiagnosticData*.
    1. En la lista **Seleccionar el nombre de directiva del Centro de eventos**, seleccione **RootManagerSharedAccessKey**.
    1. Seleccione **Aceptar**.

1. Seleccione **Guardar**.

# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)
### <a name="connect-activity-logs-to-your-event-hub"></a>Conexión de los registros de actividad al centro de eventos

1. En el menú izquierdo de Azure Portal, seleccione **Registro de actividad**.
1. Se abre la ventana **Registro de actividad**. Seleccione **Exportar a Centro de eventos**.

    ![Ventana Registro de actividad](media/ingest-data-no-code/activity-log.png)

1. Se abre la ventana **Exportar registro de actividad**:
 
    ![Ventana Exportar registro de actividad](media/ingest-data-no-code/export-activity-log.png)

1. En la ventana **Exportar registro de actividad**, realice los pasos siguientes:
      1. Seleccione su suscripción.
      1. En la lista **Regiones**, elija **Seleccionar todo**.
      1. Active la casilla **Exportar a un centro de eventos**.
      1. Elija **Select a service bus namespace** (Seleccionar un espacio de nombres del bus de servicio) para abrir el panel **Seleccionar centro de eventos**.
      1. En el panel **Seleccionar centro de eventos**, seleccione su suscripción.
      1. En la lista **Seleccionar el espacio de nombres del Centro de eventos**, seleccione *AzureMonitoringData*.
      1. En la lista **Seleccionar el nombre de directiva del Centro de eventos**, seleccione el nombre de la directiva del centro de eventos predeterminada.
      1. Seleccione **Aceptar**.
      1. En la esquina superior izquierda de la ventana, seleccione **Guardar**.
   Se creará un centro de eventos con el nombre *insights-operational-logs*.
---

### <a name="see-data-flowing-to-your-event-hubs"></a>Visualización del flujo de datos a los centros de eventos

1. Espere unos minutos hasta que se defina la conexión y haya finalizado la exportación del registro de actividad al centro de eventos. Vaya al espacio de nombres de Event Hubs para ver los centros de eventos que ha creado.

    ![Centros de eventos creados](media/ingest-data-no-code/event-hubs-created.png)

1. Vea el flujo de datos al centro de eventos:

    ![Datos del centro de eventos](media/ingest-data-no-code/event-hubs-data.png)

## <a name="connect-an-event-hub-to-azure-data-explorer"></a>Conexión de un centro de eventos a Azure Data Explorer

Ahora debe crear las conexiones de datos para sus métricas y registros de diagnóstico y registros de actividad.

### <a name="create-the-data-connection-for-diagnostic-metrics-and-logs-and-activity-logs"></a>Creación de conexiones de datos para sus métricas y registros de diagnóstico y registros de actividad

1. En el clúster de Azure Data Explorer llamado *kustodocs*, seleccione **Bases de datos** en el menú izquierdo.
1. En la ventana **Bases de datos**, seleccione la base de datos *TestDatabase*.
1. En el menú izquierdo, seleccione **Ingesta de datos**.
1. En la ventana **Ingesta de datos**, haga clic en **+ Agregar conexión de datos**.
1. En la ventana **Conexión de datos**, escriba la siguiente información:

    ![Conexión de datos del centro de eventos](media/ingest-data-no-code/event-hub-data-connection.png)

# <a name="diagnostic-metrics--diagnostic-logs"></a>[Métricas de diagnóstico y registros de diagnóstico](#tab/diagnostic-metrics+diagnostic-logs) 

1. Use la configuración siguiente en la ventana **Conexión de datos**:

    Origen de datos:

    **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | **Nombre de la conexión de datos** | *DiagnosticsLogsConnection* | Nombre de la conexión que desea crear en el Explorador de datos de Azure.|
    | **Espacio de nombres del centro de eventos** | *AzureMonitoringData* | Nombre elegido anteriormente que identifica el espacio de nombres. |
    | **Centro de eventos** | *DiagnosticData* | El centro de eventos que creó. |
    | **Grupo de consumidores** | *adxpipeline* | Grupo de consumidores de eventos definido en el centro de eventos que creó. |
    | | |

    Tabla de destino:

    Hay dos opciones para el enrutamiento: *estático* y *dinámico*. En este tutorial usará el enrutamiento estático (el predeterminado), en el que se especifican el nombre de la tabla, el formato de los datos y la asignación. Deje **Mis datos incluyen información de enrutamiento** sin seleccionar.

     **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | **Table** | *DiagnosticRawRecords* | La tabla que creó en la base de datos *TestDatabase*. |
    | **Formato de datos** | *JSON* | El formato usado en la tabla. |
    | **Asignación de columnas** | *DiagnosticRawRecordsMapping* | La asignación que creó en la base de datos *TestDatabase*, que asigna los datos JSON entrantes a los nombres de columna y los tipos de datos de la tabla *DiagnosticRawRecords*.|
    | | |

1. Seleccione **Crear**.  

# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)

1. Use la configuración siguiente en la ventana **Conexión de datos**:

    Origen de datos:

    **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | **Nombre de la conexión de datos** | *ActivityLogsConnection* | Nombre de la conexión que desea crear en el Explorador de datos de Azure.|
    | **Espacio de nombres del centro de eventos** | *AzureMonitoringData* | Nombre elegido anteriormente que identifica el espacio de nombres. |
    | **Centro de eventos** | *insights-operational-logs* | El centro de eventos que creó. |
    | **Grupo de consumidores** | *$Default* | El grupo de consumidores predeterminado. Si es necesario, puede crear un grupo de consumidores diferente. |
    | | |

    Tabla de destino:

    Hay dos opciones para el enrutamiento: *estático* y *dinámico*. En este tutorial usará el enrutamiento estático (el predeterminado), en el que se especifican el nombre de la tabla, el formato de los datos y la asignación. Deje **Mis datos incluyen información de enrutamiento** sin seleccionar.

     **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | **Table** | *ActivityLogsRawRecords* | La tabla que creó en la base de datos *TestDatabase*. |
    | **Formato de datos** | *JSON* | El formato usado en la tabla. |
    | **Asignación de columnas** | *ActivityLogsRawRecordsMapping* | La asignación que creó en la base de datos *TestDatabase*, que asigna los datos JSON entrantes a los nombres de columna y los tipos de datos de la tabla *ActivityLogsRawRecords*.|
    | | |

1. Seleccione **Crear**.  
---

## <a name="query-the-new-tables"></a>Consulta de las nuevas tablas

Ahora tiene una canalización con el flujo de datos. La ingesta a través del clúster tarda cinco minutos de forma predeterminada, por lo que los datos fluyen durante unos minutos antes de empezar la consulta.

# <a name="diagnostic-metrics"></a>[Métricas de diagnóstico](#tab/diagnostic-metrics)
### <a name="query-the-diagnostic-metrics-table"></a>Consulta de la tabla de métricas de diagnóstico

La siguiente consulta analiza los datos de duración de las consultas a partir de los registros de métricas de diagnóstico de Azure Data Explorer:

```kusto
DiagnosticMetrics
| where Timestamp > ago(15m) and MetricName == 'QueryDuration'
| summarize avg(Average)
```

Resultados de la consulta:

|   |   |
| --- | --- |
|   |  avg_Average |
|   | 00:06.156 |
| | |

# <a name="diagnostic-logs"></a>[Registros de diagnóstico](#tab/diagnostic-logs)
### <a name="query-the-diagnostic-logs-table"></a>Consulta de la tabla de registros de diagnóstico

Esta canalización produce ingestas a través de un centro de eventos. Revisará los resultados de estas ingesta.
La siguiente consulta analiza el número de ingestas acumuladas en un minuto, incluido un ejemplo de `Database`, `Table` y `IngestionSourcePath` para cada intervalo:

```kusto
DiagnosticLogs
| where Timestamp > ago(15m) and OperationName has 'INGEST'
| summarize count(), any(Database, Table, IngestionSourcePath) by bin(Timestamp, 1m)
```

Resultados de la consulta:

|   |   |
| --- | --- |
|   |  count_ | any_Database | any_Table | any_IngestionSourcePath
|   | 00:06.156 | TestDatabase | DiagnosticRawRecords | `https://rtmkstrldkereneus00.blob.core.windows.net/20190827-readyforaggregation/1133_TestDatabase_DiagnosticRawRecords_6cf02098c0c74410bd8017c2d458b45d.json.zip`
| | |

# <a name="activity-logs"></a>[Registros de actividad](#tab/activity-logs)
### <a name="query-the-activity-logs-table"></a>Consulta de la tabla de registros de actividad

La siguiente consulta analiza los datos del registro de actividad de Azure Data Explorer:

```kusto
ActivityLogs
| where OperationName == 'MICROSOFT.EVENTHUB/NAMESPACES/AUTHORIZATIONRULES/LISTKEYS/ACTION'
| where ResultType == 'Success'
| summarize avg(DurationMs)
```

Resultados de la consulta:

|   |   |
| --- | --- |
|   |  avg(DurationMs) |
|   | 768,333 |
| | |

---

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a escribir muchas más consultas sobre los datos que extrajo de Azure Data Explorer mediante el artículo [Escritura de consultas para Azure Data Explorer](write-queries.md).
* [Supervisión de las operaciones de ingesta de Azure Data Explorer mediante registros de diagnóstico](using-diagnostic-logs.md)
* [Uso de métricas para supervisar el estado del clúster](using-metrics.md)

---
title: Supervisión de las operaciones de ingesta de Azure Data Explorer mediante registros de diagnóstico
description: Aprenda a configurar registros de diagnóstico para Azure Data Explorer para supervisar las operaciones de ingesta.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/18/2019
ms.openlocfilehash: c4c7114dd53b8f77150f88cf3da5d26ca73bb93b
ms.sourcegitcommit: 2baa6a1c25ba25ef02a53b4f8d023ea821bd04ed
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2020
ms.locfileid: "81527595"
---
# <a name="monitor-azure-data-explorer-ingestion-operations-using-diagnostic-logs"></a>Supervisión de las operaciones de ingesta de Azure Data Explorer mediante registros de diagnóstico

Azure Data Explorer es un servicio de análisis de datos rápido y totalmente administrado para analizar en tiempo real grandes volúmenes de datos de que se transmiten desde aplicaciones, sitios web, dispositivos IoT, etc. Para usar Azure Data Explorer, cree primero un clúster y una o varias bases de datos en ese clúster. A continuación, ingerirá (cargará) los datos en una tabla de una base de datos para que pueda ejecutar consultas en ella. Los [registros de diagnóstico de Azure Monitor](/azure/azure-monitor/platform/diagnostic-logs-overview) proporcionan datos sobre el funcionamiento de los recursos de Azure. Azure Data Explorer usa registros de diagnóstico para extraer conclusiones sobre los errores y aciertos de la ingesta. Puede exportar los registros de operaciones a Azure Storage, Event Hub o Log Analytics para supervisar el estado de ingesta. Los registros de Azure Storage y Azure Event Hub se pueden enrutar a una tabla del clúster de Azure Data Explorer para analizarlos mejor.

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta de Azure gratuita](https://azure.microsoft.com/free/).
* Creación de un [clúster y una base de datos](create-cluster-database-portal.md).

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="set-up-diagnostic-logs-for-an-azure-data-explorer-cluster"></a>Configuración de registros de diagnóstico en un clúster de Azure Data Explorer

Los registros de diagnóstico se pueden usar para configurar la recopilación de los siguientes datos de registro:
* Operaciones de ingesta correctas: Estos registros contienen información sobre las operaciones de ingesta completadas correctamente.
* Operaciones de ingesta con error: Estos registros contienen información detallada sobre las operaciones de ingesta con error, incluidos los detalles del error. 

Los datos se archivan en una cuenta de almacenamiento, se transmiten a un centro de eventos o se envían a Log Analytics, según sus especificaciones.

### <a name="enable-diagnostic-logs"></a>Habilitar registros de diagnóstico

Los registros de diagnóstico están inhabilitados de forma predeterminada. Para habilitarlos, haga lo siguiente:

1. En [Azure Portal](https://portal.azure.com), seleccione el recurso de clúster de Azure Data Explorer que quiere supervisar.
1. En **Supervisión**, seleccione **Configuración de diagnóstico**.
  
    ![Adición de registros de diagnóstico](media/using-diagnostic-logs/add-diagnostic-logs.png)

1. Seleccione **Agregar configuración de diagnóstico**.
1. En la ventana **Configuración de diagnóstico**:
 
    ![Configuración de diagnóstico](media/using-diagnostic-logs/configure-diagnostics-settings.png) 

    1. Seleccione el **nombre** de su configuración de diagnóstico.
    1. Seleccione uno o varios destinos: una cuenta de almacenamiento, Event Hub o Log Analytics.
    1. Seleccione los registros que se van a recopilar: `SucceededIngestion` o `FailedIngestion`.
    1. Seleccione las [métricas](using-metrics.md#supported-azure-data-explorer-metrics) que se van a recopilar (opcional).  
    1. Seleccione **Guardar** para guardar la nueva configuración y las métricas de los registros de diagnóstico.

La nueva configuración se establecerá en unos minutos. Los registros aparecen en el destino de archivo configurado (cuenta de almacenamiento, Event Hub o Log Analytics). 

## <a name="diagnostic-logs-schema"></a>Esquema de registros de diagnósticos

Todos los [registros de diagnóstico de Azure Monitor comparten un esquema de nivel superior común](/azure/azure-monitor/platform/diagnostic-logs-schema). Azure Data Explorer tiene propiedades únicas para sus propios eventos. Todos los registros se almacenan en formato JSON.

### <a name="ingestion-logs-schema"></a>Esquema de registros de ingesta

Las cadenas JSON de registro incluyen elementos enumerados en la tabla siguiente:

|Nombre               |Descripción
|---                |---
|time               |Hora del informe.
|resourceId         |El identificador de recursos de Azure Resource Manager
|operationName      |Nombre de la operación: 'MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION'
|operationVersion   |Versión de esquema: "1.0" 
|category           |Categoría de la operación. `SucceededIngestion` o `FailedIngestion`. Las propiedades no son iguales para [operación correcta](#successful-ingestion-operation-log) que para [operación con error](#failed-ingestion-operation-log).
|properties         |Información detallada de la operación.

#### <a name="successful-ingestion-operation-log"></a>Registro de operaciones de ingesta correctas

**Ejemplo**:

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "SucceededIngestion",
    "properties":
    {
        "succeededOn": "2019-05-27 07:55:05.3693628",
        "operationId": "b446c48f-6e2f-4884-b723-92eb6dc99cc9",
        "database": "Samples",
        "table": "StormEvents",
        "ingestionSourceId": "66a2959e-80de-4952-975d-b65072fc571d",
        "ingestionSourcePath": "https://kustoingestionlogs.blob.core.windows.net/sampledata/events8347293.json",
        "rootActivityId": "d0bd5dd3-c564-4647-953e-05670e22a81d"
    }
}
```
**Propiedades del registro de diagnóstico de una operación correcta**

|Nombre               |Descripción
|---                |---
|succeededOn        |Hora de finalización de la ingesta
|operationId        |Identificador de operación de ingesta de Azure Data Explorer
|database           |Nombre de la base de datos de destino
|table              |Nombre de la tabla de destino.
|ingestionSourceId  |Identificador del origen de datos de ingesta
|ingestionSourcePath|Ruta de acceso del URI del blob o del origen de datos de ingesta
|rootActivityId     |Identificador de actividad

#### <a name="failed-ingestion-operation-log"></a>Registro de operaciones de ingesta con error

**Ejemplo**:

```json
{
    "time": "",
    "resourceId": "",
    "operationName": "MICROSOFT.KUSTO/CLUSTERS/INGEST/ACTION",
    "operationVersion": "1.0",
    "category": "FailedIngestion",
    "properties":
    {
        "failedOn": "2019-05-27 08:57:05.4273524",
        "operationId": "5956515d-9a48-4544-a514-cf4656fe7f95",
        "database": "Samples",
        "table": "StormEvents",
        "ingestionSourceId": "eee56f8c-2211-4ea4-93a6-be556e853e5f",
        "ingestionSourcePath": "https://kustoingestionlogs.blob.core.windows.net/sampledata/events5725592.json",
        "rootActivityId": "52134905-947a-4231-afaf-13d9b7b184d5",
        "details": "Permanent failure downloading blob. URI: ..., permanentReason: Download_SourceNotFound, DownloadFailedException: 'Could not find file ...'",
        "errorCode": "Download_SourceNotFound",
        "failureStatus": "Permanent",
        "originatesFromUpdatePolicy": false,
        "shouldRetry": false
    }
}
```

**Propiedades del registro de diagnóstico de una operación con error**

|Nombre               |Descripción
|---                |---
|failedOn           |Hora de finalización de la ingesta
|operationId        |Identificador de operación de ingesta de Azure Data Explorer
|database           |Nombre de la base de datos de destino
|table              |Nombre de la tabla de destino.
|ingestionSourceId  |Identificador del origen de datos de ingesta
|ingestionSourcePath|Ruta de acceso del URI del blob o del origen de datos de ingesta
|rootActivityId     |Identificador de actividad
|detalles            |Descripción detallada del error y el mensaje de error
|errorCode          |Código de error 
|failureStatus      |`Permanent` o `Transient`. El reintento de un error transitorio puede conseguirse.
|originatesFromUpdatePolicy|True si el error se origina desde una directiva de actualización
|shouldRetry        |True si el reintento puede conseguirse

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Ingesta y consulta de datos de supervisión en Azure Data Explorer](ingest-data-no-code.md)
* [Uso de métricas para supervisar el estado del clúster](using-metrics.md)


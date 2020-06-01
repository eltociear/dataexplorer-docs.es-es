---
title: 'Información de diagnóstico: Azure Explorador de datos'
description: En este artículo se describe la información de diagnóstico en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 505e5443e18007f41ca3fb67046df31fcbae2ba2
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84257950"
---
# <a name="diagnostic-information"></a>Información de diagnóstico

Estos comandos se pueden usar para mostrar información de diagnóstico del sistema.

* [. Mostrar clúster](#show-cluster)
* [. Mostrar diagnósticos](#show-diagnostics)
* [. Mostrar capacidad](#show-capacity)
* [. Mostrar operaciones](#show-operations)

## <a name="show-cluster"></a>. Mostrar clúster

```kusto
.show cluster
```

Devuelve un conjunto que contiene un registro para cada nodo que está activo actualmente en el clúster.  

**Resultados** 

|Columna de salida |Tipo |Descripción
|---|---|---|
|NodeId|String|Identifica el nodo. El identificador de nodo es el RoleId de Azure del nodo, si el clúster está implementado en Azure.
|Dirección|String |El punto de conexión interno usado por el clúster para las comunicaciones entre nodos
|Nombre |String |Un nombre interno para el nodo. El nombre incluye el nombre del equipo, el nombre del proceso y el identificador del proceso.
|StartTime |DateTime |Fecha y hora (en UTC) en que se inició la instancia de Kusto actual en el nodo. Este valor se puede usar para detectar si el nodo (o Kusto que se ejecuta en el nodo) se reinició recientemente.
|IsAdmin |Boolean |Si este nodo es actualmente el "líder" del clúster 
|MachineTotalMemory  |Int64 |La cantidad de RAM del nodo.
|MachineAvailableMemory  |Int64 |Cantidad de RAM que está actualmente disponible para su uso en el nodo.
|ProcessorCount  |Int32 |Número de procesadores del nodo.
|EnvironmentDescription  |string |Información adicional sobre el entorno del nodo serializado como JSON. Por ejemplo, como dominios de error o de actualización

**Ejemplo**

```kusto
.show cluster
```

NodeID|Dirección|Nombre|StartTime|IsAdmin|MachineTotalMemory|MachineAvailableMemory|ProcessorCount|EnvironmentDescription
---|---|---|---|---|---|---|---|---
Kusto. Azure. Svc_IN_1|net. TCP://100.112.150.30:23107/|Kusto. Azure. Svc_IN_4/RD000D3AB1E9BD/WaWorkerHost/3820|2016-01-15 02:00:22.6522152|True|274877435904|247797796864|16|{"UpdateDomain": 0, "FaultDomain": 0}
Kusto. Azure. Svc_IN_3|net. TCP://100.112.154.34:23107/|Kusto. Azure. Svc_IN_3/RD000D3AB1E062/WaWorkerHost/2760|2016-01-15 05:52:52.1434683|False|274877435904|258740346880|16|{"UpdateDomain": 1, "FaultDomain": 1}
Kusto. Azure. Svc_IN_2|net. TCP://100.112.128.40:23107/|Kusto. Azure. Svc_IN_2/RD000D3AB1E054/WaWorkerHost/3776|2016-01-15 07:17:18.0699790|False|274877435904|244232339456|16|{"UpdateDomain": 2, "FaultDomain": 2}
Kusto. Azure. Svc_IN_0|net. TCP://100.112.138.15:23107/|Kusto. Azure. Svc_IN_0/RD000D3AB0D6C6/WaWorkerHost/3208|2016-01-15 09:46:36.9865016|False|274877435904|238414581760|16|{"UpdateDomain": 3, "FaultDomain": 3}


## <a name="show-diagnostics"></a>. Mostrar diagnósticos

```kusto
.show diagnostics
```

Devuelve información sobre el estado de mantenimiento del clúster de Kusto.
 
**Devuelve**

|Parámetro de salida |Tipo |Descripción|
|-----------------|-----|-----------| 
|IsHealthy|Boolean|Si el clúster está en buen estado o no
|IsScaleOutRequired|Boolean|Si el clúster se debe aumentar de tamaño agregando más nodos informáticos
|MachinesTotal|Int64|El número de máquinas en el clúster
|MachinesOffline|Int64|El número de máquinas que están actualmente sin conexión
|NodeLastRestartedOn|DateTime|Última fecha y hora en que se reinició cualquier nodo del clúster
|AdminLastElectedOn|DateTime|La última propiedad de fecha y hora del rol de administrador de clústeres cambiada
|MemoryLoadFactor|Double|La cantidad de datos que contiene el clúster, con respecto a su capacidad máxima de 100,0
|ExtentsTotal|Int64|El número total de extensiones de datos que el clúster tiene actualmente, en todas las bases de datos y en todas las tablas
|Reserved|Int64|
|Reserved|Int64|
|InstancesTargetBasedOnDataCapacity|Int64|El número de instancias necesario para llevar el ClusterDataCapacityFactor por debajo de 80. Este valor solo es válido cuando todas las máquinas tienen el mismo tamaño
|TotalOriginalDataSize|Int64|Tamaño total de los datos recopilados originalmente
|TotalExtentSize|Int64|Tamaño total de los datos almacenados, después de la compresión y la indización
|IngestionsLoadFactor|Double|Porcentaje de la capacidad de ingesta del clúster que se usó. La capacidad máxima puede verse con el `.show capacity` comando
|IngestionsInProgress|Int64|El número de operaciones de ingesta realizadas actualmente
|IngestionsSuccessRate|Double|Porcentaje de operaciones de ingesta que se completaron correctamente en los 10 minutos anteriores
|MergesInProgress|Int64|El número de operaciones de combinación de extensiones que se están realizando actualmente
|BuildVersion|String|La versión de software Kusto implementada en el clúster
|BuildTime|DateTime|Fecha y hora de la versión de compilación del software de Kusto.
|ClusterDataCapacityFactor|Double|Porcentaje de la capacidad de datos del clúster utilizada. El porcentaje se calcula como SUM (datos de tamaño de extensión)/SUM (tamaño de caché de SSD).
|IsDataWarmingRequired|Boolean|Interno: si se deben ejecutar las consultas cálidos del clúster, para llevar los datos a la memoria caché de SSD local 
|DataWarmingLastRunOn|DateTime|Última fecha y hora en que se ejecutaron los datos cálidos en el clúster
|MergesSuccessRate|Double|Porcentaje de operaciones de combinación que se completaron correctamente en los 10 minutos anteriores.
|NotHealthyReason|String|Especifica el motivo por el que el clúster no tiene un estado correcto 
|IsAttentionRequired|Boolean|Si el clúster requiere la atención del equipo de operaciones
|AttentionRequiredReason|String|Especifica el motivo por el que el clúster requiere atención
|ProductVersion|String|Especifica la información del producto (rama, versión, etc.)
|FailedIngestOperations|Int64|Número de operaciones de ingesta erróneas en los 10 minutos anteriores
|FailedMergeOperations|Int64|Número de operaciones de combinación con errores en la primera hora anterior
|MaxExtentsInSingleTable|Int64|Número máximo de extensiones en la tabla (TableWithMaxExtents)
|TableWithMaxExtents|String|Tabla con el número máximo de extensiones (MaxExtentsInSingleTable)
|WarmExtentSize|Double|Tamaño total de las extensiones en la caché activa
|NumberOfDatabases|Int32|Número de bases de datos en el clúster

## <a name="show-capacity"></a>. Mostrar capacidad

```kusto
.show capacity
```

Devuelve los resultados de un cálculo para una capacidad de clúster estimada para cada recurso.
 
**Resultados**

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|Resource |String |Nombre del recurso. 
|Total |Int64 |La cantidad total de recursos, del tipo ' Resource ', que están disponibles. Por ejemplo, el número de ingesta simultáneas
|Consumida |Int64 |La cantidad de recursos del tipo ' recurso ' consumidos ahora
|Pendiente |Int64 |La cantidad de recursos restantes del tipo ' Resource '
 
**Ejemplo**

|Resource |Total |Consumida |Pendiente
|---|---|---|---
|ingesta |576 |1 |575

## <a name="show-operations"></a>. Mostrar operaciones

Este comando devuelve una tabla que contiene todas las operaciones administrativas desde que se eligió el nuevo nodo de administración.

|||
|---|---| 
|`.show` `operations`              |Devuelve todas las operaciones que el clúster está procesando o que ha procesado.
|`.show``operations` *OperationId*|Devuelve el estado de la operación de un identificador específico.
|`.show``operations` `(` *OperationId1* `,` *OperationId2* `,` ...)|Devuelve el estado de las operaciones de identificadores específicos

**Resultados**
 
|Parámetro de salida |Tipo |Descripción
|---|---|---
|id |String |Identificador de operación
|Operación |String |Alias del comando de administración
|NodeId |String |Si el comando ejecuta algo de forma remota, como DataIngestPull. El identificador de nodo contendrá el identificador del nodo remoto que se está ejecutando.
|Inicio de |DateTime |Fecha y hora (en UTC) en que se inició la operación 
|LastUpdatedOn |DateTime |Fecha y hora (en UTC) en que se actualizó la operación por última vez. La operación puede ser un paso dentro de la operación o un paso de finalización
|Duration |DateTime |Intervalo de tiempo entre LastUpdateOn e Started
|Estado |String |Estado del comando, con los valores "Ingress", "completed" o "Failed"
|Estado |String |Cadena de ayuda adicional que contiene los errores de las operaciones con errores
 
**Ejemplo**
 
|ID |Operación |Id. de nodo |Iniciado el |Última actualización el |Duration |Estado |Estado 
|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |SchemaShow | |2015-01-06 08:47:01.0000000 |2015-01-06 08:47:01.0000000 |0001-01-01 00:00:00.0000000 |Completado | 
|841fafa4-076a-4cba-9300-4836da0d9c75 |DataIngestPull |Kusto. Azure. Svc_IN_1 |2015-01-06 08:47:02.0000000 |2015-01-06 08:48:19.0000000 |0001-01-01 00:01:17.0000000 |Completado | 
|e198c519-5263-4629-a158-8d68f7a1022f |OperationsShow | |2015-01-06 08:47:18.0000000 |2015-01-06 08:47:18.0000000 |0001-01-01 00:00:00.0000000 |Completado |
|a9f287a1-f3e6-4154-ad18-b86438da0929 |ExtentsDrop | |2015-01-11 08:41:01.0000000 |0001-01-01 00:00:00.0000000 |0001-01-01 00:00:00.0000000 |InProgress |
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DataIngestPull | |2015-01-10 14:57:41.0000000 |2015-01-10 14:57:41.0000000 |0001-01-01 00:00:00.0000000 |Failed |Se modificó la colección. No se puede ejecutar la operación de enumeración |

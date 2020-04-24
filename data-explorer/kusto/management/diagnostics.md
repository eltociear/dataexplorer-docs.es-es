---
title: 'Información de diagnóstico: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe información de diagnóstico en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ae8efdf99b7ed91285e90defed7568d2a440240d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521274"
---
# <a name="diagnostic-information"></a>Información de diagnóstico

Los siguientes comandos se pueden utilizar para mostrar información de diagnóstico del sistema.

## <a name="show-cluster"></a>Clúster .show

```kusto
.show cluster
```

Devuelve un conjunto que tiene un registro por nodo actualmente activo en el clúster.  

**Resultados**

|Columna de salida |Tipo |Descripción 
|---|---|---|
|NodeId|String|Identifica el nodo (este es el RoleId de Azure del nodo si el clúster se implementa en Azure). |
|Dirección|String |El punto de conexión interno utilizado por el clúster para las comunicaciones entre nodos. 
|NOMBRE |String |Un nombre interno para el nodo (esto incluye el nombre de la máquina, el nombre del proceso y el identificador de proceso). 
|StartTime |DateTime |La fecha/hora exacta (en UTC) que comenzó la creación de instancias de Kusto actual en el nodo. Esto se puede utilizar para detectar si el nodo (o Kusto que se ejecuta en el nodo) se ha reiniciado recientemente. 
|IsAdmin |Boolean |Si este nodo es actualmente el "líder" del clúster. 
|MachineTotalMemory  |Int64 |La cantidad de RAM que tiene el nodo. 
|MachineAvailableMemory  |Int64 |La cantidad de RAM que actualmente está "disponible para su uso" en el nodo. 
|ProcessorCount  |Int32 |La cantidad de procesadores en el nodo. 
|EnvironmentDescription  |string |Información adicional sobre el entorno del nodo (por ejemplo, dominios de actualización/error), serializado como JSON. 

**Ejemplo**

```kusto
.show cluster
```

NodeId|Dirección|NOMBRE|StartTime|IsAdmin|MachineTotalMemory|MachineAvailableMemory|ProcessorCount|EnvironmentDescription
---|---|---|---|---|---|---|---|---
Kusto.Azure.Svc_IN_1|net.tcp://100.112.150.30:23107/|Kusto.Azure.Svc_IN_4/RD000D3AB1E9BD/WaWorkerHost/3820|2016-01-15 02:00:22.6522152|True|274877435904|247797796864|16|"UpdateDomain":0, "FaultDomain":0?
Kusto.Azure.Svc_IN_3|net.tcp://100.112.154.34:23107/|Kusto.Azure.Svc_IN_3/RD000D3AB1E062/WaWorkerHost/2760|2016-01-15 05:52:52.1434683|False|274877435904|258740346880|16|"UpdateDomain":1, "FaultDomain":1o
Kusto.Azure.Svc_IN_2|net.tcp://100.112.128.40:23107/|Kusto.Azure.Svc_IN_2/RD000D3AB1E054/WaWorkerHost/3776|2016-01-15 07:17:18.0699790|False|274877435904|244232339456|16|"UpdateDomain":2, "FaultDomain":2"
Kusto.Azure.Svc_IN_0|net.tcp://100.112.138.15:23107/|Kusto.Azure.Svc_IN_0/RD000D3AB0D6C6/WaWorkerHost/3208|2016-01-15 09:46:36.9865016|False|274877435904|238414581760|16|"UpdateDomain":3, "FaultDomain":3o


## <a name="show-diagnostics"></a>Diagnóstico .show

```kusto
.show diagnostics
```

Devuelve una información sobre el estado de mantenimiento del clúster de Kusto.
 
**Devuelve**

|Parámetro de salida |Tipo |Descripción|
|-----------------|-----|-----------| 
|IsHealthy|Boolean|Si el clúster se considera correcto o no.
|IsScaleOutRequired|Boolean|Si el clúster debe aumentar su tamaño (añadir más nodos informáticos). 
|MachinesTotal|Int64|El número de máquinas del clúster.
|MachinesOffline|Int64|El número de máquinas que están actualmente sin conexión (no responde).
|NodeLastRestartedOn|DateTime|La última vez que se reinicia cualquiera de los nodos del clúster.
|AdminLastElectedOn|DateTime|La última vez que ha cambiado la propiedad del rol de administrador del clúster.
|MemoryLoadFactor|Double|La cantidad de datos en poder del clúster en relación con su capacidad (que es 100,0)
|ExtentsTotal|Int64|El número total de extensiones de datos que el clúster tiene actualmente en todas las bases de datos y todas las tablas.
|Reserved|Int64|
|Reserved|Int64|
|InstancesTargetBasedOnDataCapacity|Int64| El número de instancias necesarias para que ClusterDataCapacityFactor sea inferior a 80 (válido solo cuando todas las máquinas tienen el mismo tamaño).
|TotalOriginalDataSize|Int64|Tamaño total de los datos originalmente ingeridos
|TotalExtentSize|Int64|Tamaño total de los datos almacenados (después de la compresión e indexación)
|IngestionsLoadFactor|Double|El porcentaje de utilización de la capacidad de ingesta del clúster (se puede ver mediante el comando .show capacity)
|IngestionesInProgress|Int64|El número de operaciones de ingesta que se están realizando actualmente.
|IngestionsSuccessRate|Double|El porcentaje de operaciones de ingesta que se completaron correctamente en los últimos 10 minutos.
|MergesInProgress|Int64|El número de operaciones de combinación de extensiones que se están realizando actualmente.
|BuildVersion|String|La versión de software de Kusto implementada en el clúster.
|BuildTime|DateTime|El tiempo de compilación de esta versión de software de Kusto.
|ClusterDataCapacityFactor|Double|Porcentaje de la utilización de la capacidad de datos del clúster. Se calcula como SUM(Extent Size Data) / SUM(SSD Cache Size).
|IsDataWarmingRequired|Boolean|Interno: si se deben ejecutar las consultas de calentamiento del clúster (para llevar datos a la caché SSD local). 
|DataWarmingLastRunon|DateTime|La última vez que se ejecutaron datos .warm en el clúster
|MergesSuccessRate|Double|Porcentaje de operaciones de combinación que se completaron correctamente en los últimos 10 minutos.
|NotHealthyReason|String|Cadena que especifica el motivo por el que el clúster no está en buen estado 
|IsAttentionRequired|Boolean|Si el clúster requiere la atención del equipo de Operaciones
|AtenciónRequeridoCausa|String|Cadena que especifica el motivo del clúster que requiere atención
|ProductVersion|String|Cadena con información del producto (rama, versión, etc.)
|FailedIngestOperations|Int64|Número de operaciones de ingesta fallidas en 10 minutos
|FailedMergeOperations|Int64|Número de operaciones de fusión fallidas después de 1 hora
|MaxExtentsInSingleTable|Int64|Número máximo de extensiones en la tabla (TableWithMaxExtents)
|TableWithMaxExtents|String|Tabla con el número máximo de extensiones (MaxExtentsInSingleTable)
|WarmExtentSize|Double|Tamaño total de las extensiones en la caché activa
|NumberOfBases de datos|Int32|Número de bases de datos en el clúster

## <a name="show-capacity"></a>Capacidad .show 

```kusto
.show capacity
```

Devuelve un cálculo para una capacidad de clúster estimada para cada recurso. 
 
**Resultados**

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|Resource |String |Nombre del recurso. 
|Total |Int64 |La cantidad de recursos totales de tipo 'Recurso' que están disponibles (por ejemplo, cantidad de ingestas simultáneas) 
|Consumida |Int64 |La cantidad de cuántos recursos de tipo 'Recurso' consumió en este momento 
|Pendiente |Int64 |La cantidad de recursos restantes de tipo 'Recurso' 
 
**Ejemplo**

|Resource |Total |Consumida |Pendiente 
|---|---|---|---
|Ingestión |576 |1 |575 

## <a name="show-operations"></a>.show operaciones 

Devuelve una tabla con todas las operaciones administrativas desde que se eligió el nuevo nodo Admin. 

|||
|---|---| 
|`.show` `operations`              |Devuelve todas las operaciones que el clúster ha procesado o está procesando 
|`.show``operations` *OperationId*|Devuelve el estado de la operación para un identificador específico 
|`.show``operations` `,` *OperationId2* `,` *OperationId1* OperationId1 OperationId2 ...) `(`|Devuelve el estado de las operaciones para los iDs específicos

**Resultados**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|Identificador |String |Identificador de operación. 
|Operación |String |Alias del comando Admin 
|NodeId |String |Si el comando tiene una ejecución remota (por ejemplo, DataIngestPull) - NodeId contendrá el identificador del nodo remoto en ejecución 
|StartedOn |DateTime |Fecha/hora (en UTC) cuando se ha iniciado la operación 
|LastUpdatedOn |DateTime |Fecha y hora (en UTC) cuando la operación se actualizó por última vez (puede ser un paso dentro de la operación o un paso de finalización) 
|Duration |DateTime |TimeSpan entre LastUpdateOn y StartedOn 
|State |String |Estado del comando: puede tener valores de "InProgress", "Completed" o "Failed" 
|Situación |String |Cadena de ayuda adicional que contiene errores para operaciones con errores 
 
**Ejemplo**
 
|Identificador |Operación |Id. de nodo |Empezó en |Ultima actualización en |Duration |State |Situación 
|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |SchemaShow | |2015-01-06 08:47:01.0000000 |2015-01-06 08:47:01.0000000 |0001-01-01 00:00:00.0000000 |Completed | 
|841fafa4-076a-4cba-9300-4836da0d9c75 |DataIngestPull |Kusto.Azure.Svc_IN_1 |2015-01-06 08:47:02.0000000 |2015-01-06 08:48:19.0000000 |0001-01-01 00:01:17.0000000 |Completed | 
|e198c519-5263-4629-a158-8d68f7a1022f |OperationsShow | |2015-01-06 08:47:18.0000000 |2015-01-06 08:47:18.0000000 |0001-01-01 00:00:00.0000000 |Completed | 
|a9f287a1-f3e6-4154-ad18-b86438da0929 |ExtentsDrop | |2015-01-11 08:41:01.0000000 |0001-01-01 00:00:00.0000000 |0001-01-01 00:00:00.0000000 |InProgress | 
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DataIngestPull | |2015-01-10 14:57:41.0000000 |2015-01-10 14:57:41.0000000 |0001-01-01 00:00:00.0000000 |Con error |Se modificó la colección; operación de enumeración no se puede ejecutar. 
---
title: 'Operations Management: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la administración de operaciones en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e4e373cd694de989b2bbef8058aaf1c8b3ca3025
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616480"
---
# <a name="operations-management"></a>Administración de operaciones

## <a name="show-operations"></a>. Mostrar operaciones

`.show``operations` el comando devuelve una tabla con todas las operaciones administrativas que se ejecutan y completan, que se ejecutaron en las últimas dos semanas (que actualmente es la configuración del período de retención).

**Sintaxis**

|||
|---|---| 
|`.show` `operations`              |Devuelve todas las operaciones que el clúster ha procesado o está procesando 
|`.show``operations` *OperationId*|Devuelve el estado de la operación de un identificador específico. 
|`.show``operations` `,` *OperationId1* OperationId1 OperationId2`,` ...) *OperationId2* `(`|Devuelve el estado de las operaciones de identificadores específicos

**Resultados**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|Identificador |String |Identificador de operación.
|Operación |String |Alias del comando de administración 
|NodeId |String |Si el comando tiene una ejecución remota (por ejemplo, DataIngestPull), NodeId contendrá el identificador del nodo remoto en ejecución. 
|Inicio de |DateTime |Fecha y hora (en UTC) en que se ha iniciado la operación 
|LastUpdatedOn |DateTime |Fecha y hora (en UTC) en que se actualizó por última vez la operación (puede ser un paso dentro de la operación o un paso de finalización) 
|Duration |DateTime |Intervalo de tiempo entre LastUpdateOn e Started 
|State |String |Estado del comando: puede tener valores "Ingress", "completed" o "Failed". 
|Status |String |Cadena de ayuda adicional que contiene errores para las operaciones con errores 
 
**Ejemplo**
 
|Identificador |Operación |Id. de nodo |Iniciado el |Última actualización el |Duration |State |Status 
|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |SchemaShow | |2015-01-06 08:47:01.0000000 |2015-01-06 08:47:01.0000000 |0001-01-01 00:00:00.0000000 |Completed | 
|841fafa4-076a-4cba-9300-4836da0d9c75 |DataIngestPull |Kusto. Azure. Svc_IN_1 |2015-01-06 08:47:02.0000000 |2015-01-06 08:48:19.0000000 |0001-01-01 00:01:17.0000000 |Completed | 
|e198c519-5263-4629-a158-8d68f7a1022f |OperationsShow | |2015-01-06 08:47:18.0000000 |2015-01-06 08:47:18.0000000 |0001-01-01 00:00:00.0000000 |Completed | 
|a9f287a1-f3e6-4154-ad18-b86438da0929 |ExtentsDrop | |2015-01-11 08:41:01.0000000 |0001-01-01 00:00:00.0000000 |0001-01-01 00:00:00.0000000 |InProgress | 
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DataIngestPull | |2015-01-10 14:57:41.0000000 |2015-01-10 14:57:41.0000000 |0001-01-01 00:00:00.0000000 |Con error |Se modificó la colección; es posible que la operación de enumeración no se ejecute. 

## <a name="show-operation-details"></a>. Mostrar detalles de la operación

Las operaciones pueden conservar los resultados (opcionalmente) y se pueden recuperar cuando se completa la operación mediante el `.show` `operation``details` 

**Notas:**

* No todos los comandos de control conservan sus resultados y los que normalmente lo hacen de forma predeterminada solo en ejecuciones asincrónicas ( `async` mediante la palabra clave). Busque en la documentación el comando específico y compruebe si lo hace (consulte, por ejemplo, [exportación de datos](data-export/index.md)). 
* El esquema de salida del `.show` `operations` `details` comando es el mismo esquema devuelto por la ejecución sincrónica del comando. 
* `.show` `operation` El `details` comando solo se puede invocar después de que la operación se haya completado correctamente. Use el [comando Mostrar operaciones](#show-operations)) para comprobar el estado de la operación antes de invocar este comando. 

**Sintaxis**

`.show``operation` *OperationId*`details`

**Resultados**

El resultado es diferente por cada tipo de operación y coincide con el esquema del resultado de la operación, cuando se ejecuta sincrónicamente. 

**Ejemplos**

El *OperationId* de este ejemplo es uno devuelto por una ejecución asincrónica de uno de los comandos de [exportación de datos](../management/data-export/index.md) .

```kusto 
.export 
  async 
  to csv ( 
    h@"https://storage1.blob.core.windows.net/containerName;secretKey", 
    h@"https://storage1.blob.core.windows.net/containerName2;secretKey" 
  ) 
  <| myLogs 

```

El comando de exportación asincrónica devolvió el siguiente identificador de operación:

|OperationId|
|---|
|56e51622-eb49-4d1a-b896-06a03178efcd|

Este identificador de operación se puede usar cuando el comando se ha completado para consultar los blobs exportados, como se indica a continuación: 

```kusto
.show operation 56e51622-eb49-4d1a-b896-06a03178efcd details 
```

**Resultados**

|Path|NumRecords|
|---|---|
|http://storage1.blob.core.windows.net/containerName/1_d08afcae2f044c1092b279412dcb571b.csv|10|
|http://storage1.blob.core.windows.net/containerName/2_454c0f1359e24795b6529da8a0101330.csv|15|
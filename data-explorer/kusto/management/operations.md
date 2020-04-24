---
title: 'Administración de operaciones: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la administración de operaciones en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 01f30a8d391948d5466ef76b2951d55aa6084e15
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520696"
---
# <a name="operations-management"></a>Administración de operaciones

## <a name="show-operations"></a>.show operaciones

`.show``operations` command devuelve una tabla con todas las operaciones administrativas, tanto en ejecución como completadas, que se ejecutaron en las últimas dos semanas (que actualmente es la configuración del período de retención).

**Sintaxis**

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

## <a name="show-operation-details"></a>Detalles de la operación .show

Las operaciones pueden (opcionalmente) conservar sus resultados, y estos `.show` `operation` se pueden recuperar cuando la operación se completa con el`details` 

**Notas:**

* No todos los comandos de control conservan sus resultados y los que `async` lo hacen normalmente de forma predeterminada solo en ejecuciones asincrónicas (mediante la palabra clave). Busque en la documentación el comando específico y compruebe si lo hace (véase, por [ejemplo, exportación](data-export/index.md)de datos). 
* El esquema de `.show` `operations` `details` salida del comando es el mismo esquema devuelto por la ejecución sincrónica del comando. 
* `.show` `operation` El `details` comando solo se puede invocar después de que la operación se haya completado correctamente. Utilice el [comando show operations](#show-operations)) para marcar el estado de la operación antes de invocar este comando. 

**Sintaxis**

`.show``operation` *OperationId*`details`

**Resultados**

El resultado es diferente por tipo de operación y coincide con el esquema del resultado de la operación, cuando se ejecuta sincrónicamente. 

**Ejemplos**

El *OperationId* en este ejemplo es uno devuelto de una ejecución asincrónica de uno de los comandos de exportación de [datos.](../management/data-export/index.md)

```kusto 
.export 
  async 
  to csv ( 
    h@"https://storage1.blob.core.windows.net/containerName;secretKey", 
    h@"https://storage1.blob.core.windows.net/containerName2;secretKey" 
  ) 
  <| myLogs 

```
El comando async export devolvió el siguiente identificador de operación:

|OperationId|
|---|
|56e51622-eb49-4d1a-b896-06a03178efcd|

Este identificador de operación se puede utilizar cuando el comando se ha completado para consultar los blobs exportados, como se indica a continuación: 

```
.show operation 56e51622-eb49-4d1a-b896-06a03178efcd details 
```

**Resultados**

|Path|NumRecords|
|---|---|
|http://storage1.blob.core.windows.net/containerName/1_d08afcae2f044c1092b279412dcb571b.csv|10|
|http://storage1.blob.core.windows.net/containerName/2_454c0f1359e24795b6529da8a0101330.csv|15|
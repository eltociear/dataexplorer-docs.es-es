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
ms.openlocfilehash: 40103d399feb61e994639c9447510ef90fef652d
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011472"
---
# <a name="operations-management"></a>Administración de operaciones

## <a name="show-operations"></a>. Mostrar operaciones

`.show`el `operations` comando devuelve una tabla con todas las operaciones administrativas que se ejecutan y completan, que se ejecutaron en las últimas dos semanas (que actualmente es la configuración del período de retención).

**Sintaxis**

|||
|---|---| 
|`.show` `operations`              |Devuelve todas las operaciones que el clúster está procesando u operaciones procesadas por el clúster
|`.show``operations` *OperationId*|Devuelve el estado de la operación de un identificador específico. 
|`.show``operations` `(` *OperationId1* `,` *OperationId2* `,` ...)|Devuelve el estado de las operaciones de identificadores específicos

**Resultados**
 
|Parámetro de salida |Tipo |Descripción
|---|---|---
|id |String |Identificador de operación
|Operación |String |Alias del comando de administración
|NodeId |String |Si el comando tiene una ejecución remota (por ejemplo, DataIngestPull), NodeId contendrá el identificador del nodo remoto en ejecución.
|Inicio de |DateTime |Fecha y hora (en UTC) en que se inició la operación
|LastUpdatedOn |DateTime |Fecha y hora (en UTC) en que se actualizó por última vez la operación (puede ser un paso dentro de la operación o un paso de finalización)
|Duration |DateTime |Intervalo de tiempo entre LastUpdateOn e Started
|State |String |Estado del comando: puede tener valores "Ingress", "completed" o "Failed".
|Estado |String |Cadena de ayuda adicional que contiene errores de operaciones con error
 
**Ejemplo**
 
|id |Operación |Id. de nodo |Iniciado el |Última actualización el |Duration |State |Estado 
|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |SchemaShow | |2015-01-06 08:47:01.0000000 |2015-01-06 08:47:01.0000000 |0001-01-01 00:00:00.0000000 |Completado |
|841fafa4-076a-4cba-9300-4836da0d9c75 |DataIngestPull |Kusto. Azure. Svc_IN_1 |2015-01-06 08:47:02.0000000 |2015-01-06 08:48:19.0000000 |0001-01-01 00:01:17.0000000 |Completado |
|e198c519-5263-4629-a158-8d68f7a1022f |OperationsShow | |2015-01-06 08:47:18.0000000 |2015-01-06 08:47:18.0000000 |0001-01-01 00:00:00.0000000 |Completado |
|a9f287a1-f3e6-4154-ad18-b86438da0929 |ExtentsDrop | |2015-01-11 08:41:01.0000000 |0001-01-01 00:00:00.0000000 |0001-01-01 00:00:00.0000000 |InProgress |
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DataIngestPull | |2015-01-10 14:57:41.0000000 |2015-01-10 14:57:41.0000000 |0001-01-01 00:00:00.0000000 |Failed |Se modificó la colección. Es posible que la operación de enumeración no se ejecute.

## <a name="show-operation-details"></a>. Mostrar detalles de la operación

Las operaciones pueden conservar los resultados (opcionalmente) y se pueden recuperar los resultados cuando se completa la operación, mediante `.show` `operation` `details` .

> [!NOTE]
> No todos los comandos de control conservan sus resultados. Estos comandos, normalmente lo hacen de forma predeterminada solo en ejecuciones asincrónicas, mediante la `async` palabra clave. Para obtener más información, consulte la documentación específica del comando. Por ejemplo, vea [exportación de datos](data-export/index.md)).
> El esquema de salida del `.show` `operations` `details` comando es el mismo esquema devuelto por la ejecución sincrónica del comando.
> El `.show` `operation` `details` comando solo se puede invocar después de que la operación se haya completado correctamente. Use el [comando Mostrar operaciones](#show-operations)) para comprobar el estado de la operación antes de ejecutar este comando.

**Sintaxis**

`.show``operation` *OperationId*`details`

**Resultados**

El resultado es diferente por cada tipo de operación y coincide con el esquema del resultado de la operación, cuando se ejecuta sincrónicamente.

**Ejemplos**

El *OperationId* en el ejemplo, devuelve de una ejecución asincrónica de uno de los comandos de [exportación de datos](../management/data-export/index.md) .

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

Este identificador de operación se puede usar cuando el comando se ha completado para consultar los blobs exportados. 

```kusto
.show operation 56e51622-eb49-4d1a-b896-06a03178efcd details 
```

**Resultados**

|Ruta de acceso|NumRecords|
|---|---|
|http://storage1.blob.core.windows.net/containerName/1_d08afcae2f044c1092b279412dcb571b.csv|10|
|http://storage1.blob.core.windows.net/containerName/2_454c0f1359e24795b6529da8a0101330.csv|15|

---
title: 'Errores de ingesta: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los errores de ingesta en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/20/2019
ms.openlocfilehash: 699fd01e9a284179bf2749b58581392d2170f0ca
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520951"
---
# <a name="ingestion-failures"></a>Fallos en la ingesta

## <a name="show-ingestion-failures"></a>Fallos de ingestión de .show

Este comando devuelve un conjunto de resultados que incluye todos los errores de ingesta que se encontraron durante la ejecución de comandos de control de ingesta de [datos.](data-ingestion/index.md)

*Notas*: 
- Los errores de ingesta que se encuentran durante otras partes del flujo de ingesta (por ejemplo, antes de que los comandos de control de ingesta de datos se envíen al servicio Motor de datos de Kusto) no aparecerán en el conjunto de resultados de este comando.
- Para los errores de supervisión que se producen en los flujos que implican la [ingesta en cola,](../api/netfx/about-kusto-ingest.md#queued-ingestion)se recomienda seguir [esta guía.](../api/netfx/kusto-ingest-client-status.md)

**Sintaxis**

|||
|---|---| 
|`.show` `ingestion` `failures`                                       |Devuelve todos los errores de ingesta registrados  
|`.show` `ingestion` `failures` <code>&#124;</code> `where` ...       |Devuelve un conjunto filtrado de errores de ingesta
|`.show``ingestion` `failures` *OperationId* OperationId `with(OperationId="``")` |Devuelve un error de ingesta para un identificador de operación específico

**Resultados**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|OperationId |String |Identificador de operación. Este parámetro se puede utilizar para ver detalles adicionales de la operación ejecutando el comando [.show operations](operations.md) 
|Base de datos |String |Base de datos en la que se encontró el error
|Tabla |String |Tabla sobre la cual se encontró el fallo
|FailedOn |DateTime |Fecha/hora (en UTC) cuando se registró el error 
|IngestionSourcePath |String |Identifica el origen de la ingesta (normalmente, un URI de blob de Azure) 
|Detalles |String |Detalles del error. Proporciona información sobre la causa raíz del error de ingestión real
|FailureKind |String |Tipo de fallo (Permanente/Transitorio)
|RootActivityId |String |ID de actividad raíz.
|OperationKind |String |El tipo de operación de ingesta (fase) durante la cual se registró el error
|OriginatesFromUpdatePolicy |Boolean | Indica si el error se registró al ejecutar una directiva de [actualización](update-policy.md)
 
**Ejemplo**
 
|OperationId |Base de datos |Tabla |FailedOn |IngestionSourcePath |Detalles |FailureKind |RootActivityId |OperationKind |OriginatesFromUpdatePolicy
|--|--|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |DB1 |Table1 |2017-02-14 22:25:03.1147331 |... Url... |Stream with id '*****.csv' tiene un formato Csv mal formado |Permanente |3c883942-e446-4999-9b00-d4c664f06ef6 |DataIngestPull | 0
|841fafa4-076a-4cba-9300-4836da0d9c75 |DB1 |Table1 |2017-02-14 22:34:11.2565943 |... Url... |Stream with id '*****.csv' tiene un formato Csv mal formado |Permanente |48571bdb-b714-4f32-8ddc-4001838a956c |DataIngestPull | 0
|e198c519-5263-4629-a158-8d68f7a1022f |DB1 |Table1 |2017-02-14 22:34:44.5824741 |... Url... |Stream with id '*****.csv' tiene un formato Csv mal formado |Permanente |5e31ab3c-e2c7-489a-827e-e89d2d691ec4 |DataIngestPull | 0
|a9f287a1-f3e6-4154-ad18-b86438da0929 |DB1 |Table1 |2017-02-14 22:36:26.5525250 |... Url... |Error desconocido: se ha producido una excepción de tipo 'System.Exception' |Transitorio |9b7bb017-471e-48f6-9c96-d16fcf938d2a |DataIngestPull | 0
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DB1 |Table1 |2017-02-14 23:52:31.5460071 |... Url... |Error al descargar el blob: el cliente no pudo finalizar la operación dentro del tiempo de espera especificado |Permanente |21fa0dd6-cd7d-4493-b6f7-78916ce0d617 |DataIngestPull | 0
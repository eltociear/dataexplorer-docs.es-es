---
title: 'Errores de ingesta: Azure Explorador de datos'
description: En este artículo se describen los errores de ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/20/2019
ms.openlocfilehash: 7684ea11b03113051580e3e19aef0d9ac3f13585
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011489"
---
# <a name="ingestion-failures"></a>Errores de ingesta

## <a name="show-ingestion-failures"></a>. Mostrar errores de ingesta


Este comando devuelve un conjunto de resultados que incluye los errores de ingesta que se producen cuando se ejecutan los [comandos de control de ingesta de datos](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands) .


> [!NOTE]
> Los errores de ingesta que se producen durante otras partes del flujo de ingesta no aparecerán en el conjunto de resultados de este comando. Este error puede producirse, por ejemplo, antes de que los comandos de control de ingesta de datos se envíen al servicio del motor de datos Kusto.

> Para obtener más información sobre los errores de supervisión que se producen en flujos que implican la [ingesta en cola](../api/netfx/about-kusto-ingest.md#queued-ingestion), vea [esta guía](../api/netfx/kusto-ingest-client-status.md).

**Sintaxis**

|||
|---|---| 
|`.show` `ingestion` `failures`                                       |Devuelve todos los errores de ingesta registrados  
|`.show` `ingestion` `failures` <code>&#124;</code> `where` ...       |Devuelve un conjunto filtrado de errores de ingesta.
|`.show``ingestion` `failures` `with(OperationId="` *OperationId*`")` |Devuelve errores de ingesta para un identificador de operación específico

**Resultados**
 
|Parámetro de salida           |Tipo     |Descripción                                                                              |
|---------------------------|---------|-----------------------------------------------------------------------------------------|
|OperationId                |String   |Identificador de la operación que se puede usar para ver detalles adicionales de la operación a través de la <br> [. Mostrar operaciones](operations.md) (comando) </br> 
|Base de datos                   |String   |Base de datos en la que se produjo el error
|Tabla                      |String   |Tabla en la que se produjo el error
|Con errores                   |DateTime |Fecha y hora (en UTC) en que se registró el error 
|IngestionSourcePath        |String   |Identifica el origen de la ingesta (normalmente, un URI de BLOB de Azure) 
|Detalles                    |String   |Detalles del error. Proporciona información sobre la causa principal del error de ingesta real.
|FailureKind                |String   |Tipo de error (permanente/transitorio)
|RootActivityId             |String   |IDENTIFICADOR de actividad raíz.
|OperationKind              |String   |El tipo de operación de ingesta (fase) durante el que se registró el error
|OriginatesFromUpdatePolicy |Booleano | Indica si el error se registró al ejecutar una [Directiva de actualización](update-policy.md)
 
**Ejemplo**
 
|OperationId |Base de datos |Tabla |Con errores |IngestionSourcePath |Detalles |FailureKind |RootActivityId |OperationKind |OriginatesFromUpdatePolicy
|--|--|--|--|--|--|--|--|--|--
|3827def6-0773-4f2a-859e-c02cf395deaf |DB1 |Table1 |2017-02-14 22:25:03.1147331 |... Dirección URL... |La secuencia con el identificador ' * * * * *. csv ' tiene un formato CSV con formato incorrecto * |Permanente |3c883942-e446-4999-9b00-d4c664f06ef6 |DataIngestPull | 0
|841fafa4-076a-4cba-9300-4836da0d9c75 |DB1 |Table1 |2017-02-14 22:34:11.2565943 |... Dirección URL... |La secuencia con el identificador ' * * * * *. csv ' tiene un formato CSV con formato incorrecto * |Permanente |48571bdb-b714-4f32-8ddc-4001838a956c |DataIngestPull | 0
|e198c519-5263-4629-a158-8d68f7a1022f |DB1 |Table1 |2017-02-14 22:34:44.5824741 |... Dirección URL... |La secuencia con el identificador ' * * * * *. csv ' tiene un formato CSV con formato incorrecto * |Permanente |5e31ab3c-e2c7-489a-827e-e89d2d691ec4 |DataIngestPull | 0
|a9f287a1-f3e6-4154-ad18-b86438da0929 |DB1 |Table1 |2017-02-14 22:36:26.5525250 |... Dirección URL... |Error desconocido: se produjo una excepción de tipo ' System. Exception ' |Transitorio |9b7bb017-471e-48f6-9c96-d16fcf938d2a |DataIngestPull | 0
|9edb3ecc-f4b4-4738-87e1-648eed2bd998 |DB1 |Table1 |2017-02-14 23:52:31.5460071 |... Dirección URL... |No se pudo descargar el BLOB: el cliente no pudo finalizar la operación dentro del tiempo de espera especificado |Permanente |21fa0dd6-cd7d-4493-b6f7-78916ce0d617 |DataIngestPull | 0

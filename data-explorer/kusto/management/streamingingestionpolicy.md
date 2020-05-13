---
title: 'Directiva de ingesta de streaming: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la Directiva de ingesta de streaming en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2020
ms.openlocfilehash: a0141482e3714ed710dbdc6fa00e3b8b396e77e6
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373306"
---
# <a name="streaming-ingestion-policy"></a>Directiva de ingesta de streaming

## <a name="streaming-ingestion-target-scenario"></a>Escenario de destino de ingesta de streaming

La ingesta de streaming está destinada a escenarios que requieren una latencia baja con un tiempo de ingesta de menos de 10 segundos de datos de volumen variado. Se usa para optimizar el procesamiento operativo de muchas tablas, en una o varias bases de datos, donde el flujo de datos en cada tabla es relativamente pequeño (pocos registros por segundo), pero el volumen de ingesta de datos global es alto (miles de registros por segundo).

Use la ingesta clásica (masiva) en lugar de la ingesta de streaming cuando la cantidad de datos crezca más de 1 MB por segundo y tabla. 

* Para obtener información sobre cómo implementar esta característica, consulte [ingesta de streaming](../../ingest-data-streaming.md).
* Para obtener información sobre los comandos de control de ingesta de streaming, consulte [los comandos de control que se usan para administrar la Directiva de ingesta de streaming](../management/streamingingestion-policy.md) .

## <a name="streaming-ingestion-policy-definition"></a>Definición de directiva de ingesta de streaming

La Directiva de ingesta de streaming se puede definir en una tabla o en una base de datos. La definición de esta directiva en el nivel de base de datos aplica la misma configuración a todas las tablas existentes y futuras de la base de datos. Si se establece la Directiva de ingesta de transmisión por secuencias en los niveles de tabla y de base de datos, la configuración de nivel de tabla tiene prioridad.

> [!NOTE]
> Si una tabla no obtiene directamente la ingesta de streaming, pero solo a través de una directiva de actualización, no se tiene que definir ninguna directiva de ingesta de streaming en esta tabla. 

La Directiva de ingesta de streaming establece el número máximo de almacenes de filas en los que se distribuirán los datos de streaming de la tabla. La distribución es necesaria para la compatibilidad con la velocidad de datos y la disponibilidad.

## <a name="setting-the-number-of-row-stores"></a>Establecer el número de almacenes de filas

Es necesario definir el número de almacenes de filas establecido en la Directiva de ingesta de transmisión por secuencias. Este número debe basarse en la velocidad de transmisión de datos por tabla (la estimación aproximada es suficiente).
El número mínimo recomendado de almacenes de filas para cualquier tabla es cuatro. El número máximo admitido de almacenes de filas es 64.
Cuanto mayor sea la velocidad de datos de streaming para la tabla, mayor será el número necesario de almacenes de filas necesarios en la Directiva de ingesta de streaming asociada.
Use la tabla siguiente para la configuración recomendada (si tiene dudas sobre cómo usar un número mayor):

|Velocidad de datos de streaming por hora máxima estimada (por tabla)|Número de almacenes de filas|
|----------|------|
|< 1 GB/hora |4|
|1-2 GB/hora |4-8|
|2-3 GB/hora |8-12|
|3-4 GB/hora |12-16|
| > 4 GB/h |

 Para obtener más consejos sobre esto, abra una [incidencia de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) .

Para una latencia de consulta óptima, el número de almacenes de filas definido por tabla no debe superar significativamente la recomendación anterior.

> [!NOTE]
> Al establecer la Directiva de ingesta de streaming para la base de datos, asigne el número de almacenes de filas que se necesitan para la tabla con la velocidad de datos más alta. 
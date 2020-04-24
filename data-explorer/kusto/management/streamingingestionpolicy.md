---
title: 'Directiva de ingesta de streaming: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de ingesta de streaming en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2020
ms.openlocfilehash: ef4199e0cd8c1e14839e5508a9c6a0d793de0cc0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519608"
---
# <a name="streaming-ingestion-policy"></a>Directiva de ingesta de streaming

## <a name="streaming-ingestion-target-scenario"></a>Escenario de destino de ingesta de streaming

La ingesta de streaming está destinada a escenarios que requieren una latencia baja con un tiempo de ingesta de menos de 10 segundos de datos de volumen variado. Se utiliza para optimizar el procesamiento operativo de muchas tablas, en una o más bases de datos, donde el flujo de datos en cada tabla es relativamente pequeño (pocos registros por segundo), pero el volumen de ingesta de datos general es alto (miles de registros por segundo).

Use la ingesta clásica (masiva) en lugar de la ingesta de streaming cuando la cantidad de datos crezca más de 1 MB por segundo y tabla. 

* Para obtener información sobre cómo implementar esta característica, consulte [ingesta](https://docs.microsoft.com/azure/data-explorer/ingest-data-streaming)de streaming .
* Para obtener información sobre los comandos de control de ingesta de streaming, consulte Comandos de control que se utilizan para administrar la directiva de [ingesta](../management/streamingingestion-policy.md) de streaming

## <a name="streaming-ingestion-policy-definition"></a>Definición de directiva de ingesta de streaming

La directiva de ingesta de streaming se puede definir en una tabla o una base de datos. La definición de esta directiva en el nivel de base de datos aplica la misma configuración a todas las tablas existentes y futuras de la base de datos. Si la directiva de ingesta de streaming se establece en los niveles de tabla y base de datos, la configuración de nivel de tabla tiene prioridad.

> [!NOTE]
> Si una tabla no obtiene la ingesta de streaming directamente, pero solo a través de una directiva de actualización, no es posible definir ninguna directiva de ingesta de streaming en esta tabla. 

La directiva de ingesta de streaming establece el número máximo de almacenes de filas en los que se distribuirán los datos de streaming de la tabla. La distribución es necesaria tanto para la disponibilidad como para la compatibilidad con la velocidad de datos.

## <a name="setting-the-number-of-row-stores"></a>Establecer el número de almacenes de filas

Es necesario definir el número de almacenes de filas establecidos en la directiva de ingesta de streaming. Este número debe basarse en la velocidad de datos de streaming por tabla (la estimación aproximada es suficiente).
El número mínimo recomendado de almacenes de filas para cualquier tabla es cuatro. El número máximo admitido de almacenes de filas es 64.
Cuanto mayor sea la velocidad de datos de streaming para la tabla, mayor será el número necesario de almacenes de filas necesarios en la directiva de ingesta de streaming asociada.
Utilice la siguiente tabla para los ajustes recomendados (en caso de duda, utilice un número mayor):

|Velocidad de datos de streaming por hora máxima estimada (por tabla)|Número de almacenes de fila|
|----------|------|
|< 1 Gb/h |4|
|1 - 2 GB/h |4-8|
|2 - 3 GB/h |8-12|
|3 - 4 GB/h |12-16|
| > 4 GB/h |

 Para obtener más consejos al respecto, abra un ticket de [soporte](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

Para una latencia de consulta óptima, el número de almacenes de filas definidos por tabla no debería superar significativamente la recomendación anterior.

> [!NOTE]
> Al establecer la directiva de ingesta de streaming para la base de datos, asigne el número de almacenes de filas necesarios para la tabla con la velocidad de datos más alta. 
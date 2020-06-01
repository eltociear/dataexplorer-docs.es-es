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
ms.openlocfilehash: 312e6cad984491b39bd9a77f0b34d142798e922e
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84258001"
---
# <a name="streaming-ingestion-policy"></a>Directiva de ingesta de streaming

## <a name="streaming-ingestion-target-scenario"></a>Escenario de destino de ingesta de streaming

La ingesta de streaming está destinada a escenarios que requieren una latencia baja, con un tiempo de ingesta de menos de 10 segundos para diversos datos de volumen. Se usa para optimizar el procesamiento operativo de muchas tablas, en una o varias bases de datos, donde el flujo de datos en cada tabla es relativamente pequeño (algunos registros por segundo), pero el volumen de ingesta de datos global es alto (miles de registros por segundo).

Use la ingesta clásica (masiva) en lugar de la ingesta de streaming cuando la cantidad de datos crezca a más de 4 GB por hora por tabla. 

* Para obtener información sobre cómo implementar esta característica, consulte [ingesta de streaming](../../ingest-data-streaming.md).
* Para obtener información sobre los comandos de control de ingesta de streaming, consulte [comandos de control usados para administrar la Directiva de ingesta de streaming](streamingingestion-policy.md) .
 
## <a name="streaming-ingestion-policy-definition"></a>Definición de directiva de ingesta de streaming

La Directiva de ingesta de transmisión por secuencias contiene las siguientes propiedades:

* **IsEnabled**:
  * define el estado de la funcionalidad de ingesta de streaming para la tabla o base de datos
  * obligatorio, sin valor predeterminado, debe establecerse explícitamente en *true* o *false* .
* **HintAllocatedRate**:
  * Si se establece, proporciona una sugerencia sobre el volumen de datos por hora en gigabytes que se espera para la tabla. Esta sugerencia ayuda al sistema a ajustar la cantidad de recursos que se asignan a una tabla para admitir la ingesta de streaming.
  * valor predeterminado *null* (no establecido)

Para habilitar la ingesta de streaming en una tabla, defina la Directiva de ingesta de streaming con *IsEnabled* establecido en *true*. Esta definición se puede establecer en una tabla o en la base de datos.
La definición de esta directiva en el nivel de base de datos aplica la misma configuración a todas las tablas existentes y futuras de la base de datos. Si la Directiva de ingesta de streaming se establece en los niveles de tabla y base de datos, la configuración de nivel de tabla tiene prioridad. Esta configuración significa que la ingesta de streaming puede estar habilitada generalmente para la base de datos, pero se ha deshabilitado específicamente para determinadas tablas, o viceversa.

> [!NOTE]
> Si una tabla no obtiene directamente la ingesta de streaming, pero solo a través de una directiva de actualización, no se tiene que definir ninguna directiva de ingesta de streaming en esta tabla.


## <a name="set-the-data-rate-hint"></a>Establecer la sugerencia de velocidad de datos
La Directiva de ingesta de streaming puede proporcionar una sugerencia sobre el volumen de datos por hora esperado para la tabla. Esta sugerencia ayudará al sistema a ajustar la cantidad de recursos asignados para esta tabla a fin de admitir la ingesta de streaming.
Establezca la sugerencia si la velocidad de entrada de datos de streaming en la tabla será superior a 1 GB/hora.
Si establece _HintAllocatedRate_ en la Directiva de ingesta de streaming para la base de datos, establézcala en la tabla con la velocidad de datos más alta esperada. No se recomienda establecer la sugerencia efectiva para una tabla en un valor mucho mayor que la velocidad de datos máxima esperada por hora. Este valor puede tener un efecto adverso en el rendimiento de las consultas.

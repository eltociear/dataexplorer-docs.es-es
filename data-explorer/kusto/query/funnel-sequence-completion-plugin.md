---
title: 'complemento de funnel_sequence_completion: Azure Explorador de datos'
description: En este artículo se describe funnel_sequence_completion complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/16/2020
ms.openlocfilehash: 57cceb2fabb16956090430161b98c1287efdef97
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227330"
---
# <a name="funnel_sequence_completion-plugin"></a>complemento funnel_sequence_completion

Calcula el embudo de los pasos de la secuencia completada en la comparación de distintos períodos de tiempo.

```kusto
T | evaluate funnel_sequence_completion(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, state_column, dynamic(['S1', 'S2', 'S3']), dynamic([10m, 30min, 1h]))
```

**Sintaxis**

*T* `| evaluate` `funnel_sequence_completion(` *IdColumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *Step* `,` *StateColumn* `,` *Sequence* `,` *MaxSequenceStepWindows*`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColum*: la referencia de columna debe estar presente en la expresión de origen.
* *TimelineColumn*: la referencia de columna que representa la escala de tiempo debe estar presente en la expresión de origen.
* *Start*: valor constante escalar del período de inicio del análisis.
* *End*: valor constante escalar del período de finalización del análisis.
* *Paso*: valor constante escalar del período de paso de análisis (bin).
* *StateColumn*: la referencia de columna que representa el estado debe estar presente en la expresión de origen.
* *Sequence*: matriz dinámica constante con los valores de secuencia (los valores se buscan en `StateColumn` ).
* *MaxSequenceStepWindows*: matriz dinámica escalar constante con los valores del intervalo de tiempo máximo permitido entre el primer y el último paso secuencial de la secuencia. Cada ventana (punto) de la matriz genera un resultado de análisis de embudo.

**Devuelve**

Devuelve una sola tabla útil para crear un diagrama de embudo para la secuencia analizada:

* `TimelineColumn`: ventana de tiempo analizada
* `StateColumn`: el estado de la secuencia.
* `Period`: período máximo (ventana) permitido para completar los pasos de la secuencia de embudo medida desde el primer paso de la secuencia. Cada valor de *MaxSequenceStepWindows* genera un análisis de embudo con un período independiente. 
* `dcount`: recuento distintivo de en período de `IdColumn` tiempo que se ha pasado del primer estado de secuencia al valor de `StateColumn` .

**Ejemplos**

### <a name="exploring-storm-events"></a>Exploración de eventos de Storm 

La siguiente consulta comprueba el embudo de finalización de la secuencia: `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` en la hora "global" de 1hour, 4horas, 1Day. 

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let _start = datetime(2007-01-01);
let _end =  datetime(2008-01-01);
let _windowSize = 365d;
let _sequence = dynamic(['Hail', 'Tornado', 'Thunderstorm Wind']);
let _periods = dynamic([1h, 4h, 1d]);
StormEvents
| evaluate funnel_sequence_completion(EpisodeId, StartTime, _start, _end, _windowSize, EventType, _sequence, _periods) 
```

|`StartTime`|`EventType`|`Period`|`dcount`|
|---|---|---|---|
|2007-01-01 00:00:00.0000000|Granizo|01:00:00|2877|
|2007-01-01 00:00:00.0000000|Tornado|01:00:00|208|
|2007-01-01 00:00:00.0000000|Viento de tormenta|01:00:00|87|
|2007-01-01 00:00:00.0000000|Granizo|04:00:00|2877|
|2007-01-01 00:00:00.0000000|Tornado|04:00:00|231|
|2007-01-01 00:00:00.0000000|Viento de tormenta|04:00:00|141|
|2007-01-01 00:00:00.0000000|Granizo|1,00:00:00|2877|
|2007-01-01 00:00:00.0000000|Tornado|1,00:00:00|244|
|2007-01-01 00:00:00.0000000|Viento de tormenta|1,00:00:00|155|

Descripción de los resultados:  
El resultado es de tres embudos (para períodos: una hora, 4 horas y un día). Para cada paso de embudo, se muestran varios recuentos distintivos de. Puede ver que se proporciona más tiempo para completar toda la secuencia de `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` , el valor más alto `dcount` se obtiene. En otras palabras, había más repeticiones de la secuencia que alcanza el paso de embudo.

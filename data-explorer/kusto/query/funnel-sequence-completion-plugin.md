---
title: funnel_sequence_completion complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe funnel_sequence_completion complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/16/2020
ms.openlocfilehash: 7f168841db2df47e4e3a192b75585a1fe4d83718
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514780"
---
# <a name="funnel_sequence_completion-plugin"></a>complemento funnel_sequence_completion

Calcula el embudo de los pasos de secuencia completados al comparar diferentes períodos de tiempo.

```kusto
T | evaluate funnel_sequence_completion(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, state_column, dynamic(['S1', 'S2', 'S3']), dynamic([10m, 30min, 1h]))
```

**Sintaxis**

*T* `| evaluate` `,` *Start* `,` *End* `,` *Step* `,` *StateColumn* `,` *Sequence* `,` *IdColumn* `,` *TimelineColumn* *MaxSequenceStepWindows* IdColumn TimelineColumn Start End Step StateColumn Sequence MaxSequenceStepWindows `funnel_sequence_completion(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColum*: referencia de columna, debe estar presente en la expresión de origen
* *TimelineColumn*: referencia de columna que representa la línea de tiempo, debe estar presente en la expresión de origen
* *Inicio*: valor constante escalar del período de inicio del análisis
* *End*: valor constante escalar del período final del análisis
* *Paso*: valor constante escalar del período de paso de análisis (bin) 
* *StateColumn*: referencia de columna que representa el estado, debe estar presente en la expresión de origen
* *Secuencia*: una matriz dinámica constante con los `StateColumn`valores de secuencia (los valores se buscan en )
* *MaxSequenceStepWindows*: matriz dinámica constante escalar con los valores del intervalo de tiempo máximo permitido entre el primer y el último paso secuencial de la secuencia, cada ventana (período) de la matriz genera un resultado de análisis de embudo de conversión

**Devuelve**

Devuelve una sola tabla útil para construir un diagrama de embudo de conversión para la secuencia analizada:

* TimelineColumn: la ventana de tiempo analizada
* `StateColumn`: el estado de la secuencia.
* Período: el período máximo (ventana) permitido para completar los pasos en la secuencia de embudo medida desde el primer paso de la secuencia. Cada valor de *MaxSequenceStepWindows* genera un análisis de embudo de conversión con un período independiente. 
* dcount: recuento `IdColumn` distinto de la ventana de tiempo que `StateColumn`pasó del estado de la primera secuencia al valor de .

**Ejemplos**

### <a name="exploring-storm-events"></a>Explorar eventos de tormenta 

La siguiente consulta comprueba el embudo de finalización de la `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` secuencia: en tiempo "general" de 1 hora, 4 horas, 1 día. 

```kusto
let _start = datetime(2007-01-01);
let _end =  datetime(2008-01-01);
let _windowSize = 365d;
let _sequence = dynamic(['Hail', 'Tornado', 'Thunderstorm Wind']);
let _periods = dynamic([1h, 4h, 1d]);
StormEvents
| evaluate funnel_sequence_completion(EpisodeId, StartTime, _start, _end, _windowSize, EventType, _sequence, _periods) 
```

|StartTime|EventType|Período|dcount|
|---|---|---|---|
|2007-01-01 00:00:00.0000000|Granizo|01:00:00|2877|
|2007-01-01 00:00:00.0000000|Tornado|01:00:00|208|
|2007-01-01 00:00:00.0000000|Viento de tormenta|01:00:00|87|
|2007-01-01 00:00:00.0000000|Granizo|04:00:00|2877|
|2007-01-01 00:00:00.0000000|Tornado|04:00:00|231|
|2007-01-01 00:00:00.0000000|Viento de tormenta|04:00:00|141|
|2007-01-01 00:00:00.0000000|Granizo|1.00:00:00|2877|
|2007-01-01 00:00:00.0000000|Tornado|1.00:00:00|244|
|2007-01-01 00:00:00.0000000|Viento de tormenta|1.00:00:00|155|

Comprender los resultados:  
El resultado que 3 embudos (para períodos: 1 hora, 4 horas y 1 día), mientras que para cada paso de embudo se muestra un número de recuento distinto de EpisodeId. Puede ver que cuanto más tiempo se da `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` para `dcount` completar toda la secuencia del valor más alto (lo que significa más apariciones de la secuencia que alcanza el paso del embudo).

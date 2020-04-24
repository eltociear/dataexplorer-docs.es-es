---
title: funnel_sequence complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe funnel_sequence complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a0543d0083fd90d5ee3e6e94cf1ee203f6cbd50d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514746"
---
# <a name="funnel_sequence-plugin"></a>complemento funnel_sequence

Calcula el recuento distinto de usuarios que han tomado una secuencia de estados y la distribución de los estados anteriores/siguientes que han llevado a / fueron seguidos por la secuencia. 

```kusto
T | evaluate funnel_sequence(id, datetime_column, startofday(ago(30d)), startofday(now()), 10m, 1d, state_column, dynamic(['S1', 'S2', 'S3']))
```

**Sintaxis**

*T* `| evaluate` `,` *Start* `,` *End* `,` *Step* *Sequence* *IdColumn* `,` *TimelineColumn* *StateColumn* *MaxSequenceStepWindow*IdColumn TimelineColumn Start End MaxSequenceStepWindow , Step , StateColumn , Sequence `funnel_sequence(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColum*: referencia de columna, debe estar presente en la expresión de origen
* *TimelineColumn*: referencia de columna que representa la línea de tiempo, debe estar presente en la expresión de origen
* *Inicio*: valor constante escalar del período de inicio del análisis
* *End*: valor constante escalar del período final del análisis
* *MaxSequenceStepWindow*: valor constante escalar del intervalo de tiempo máximo permitido entre 2 pasos secuenciales en la secuencia
* *Paso*: valor constante escalar del período de paso de análisis (bin)
* *StateColumn*: referencia de columna que representa el estado, debe estar presente en la expresión de origen
* *Secuencia*: una matriz dinámica constante con los `StateColumn`valores de secuencia (los valores se buscan en )

**Devuelve**

Devuelve 3 tablas de salida, útiles para construir un diagrama de sankey para la secuencia analizada:

* Tabla #1 - prev-sequence-next dcount TimelineColumn: la ventana de tiempo analizada anterior: el estado anterior (puede estar vacío si hubiera algún usuario que solo tuviera eventos para la secuencia buscada, pero no cualquier evento anterior a ella). 
    siguiente: el siguiente estado (puede estar vacío si hubiera usuarios que solo tuvieran eventos para la secuencia buscada, pero no cualquier evento que la siguiera). 
    dcount: recuento <IdColumn> distinto de en la ventana de <Sequence> tiempo que en transición [prev] --> --> [siguiente]. 
    muestras: una matriz de <IdColumn>identificadores (desde ) correspondientes a la secuencia de la fila (se devuelve un máximo de 128 identificadores). 

* Tabla #2 - prev-sequence dcount TimelineColumn: la ventana de tiempo analizada anterior: el estado anterior (puede estar vacío si hubiera algún usuario que solo tenía eventos para la secuencia buscada, pero no ningún evento anterior). 
    dcount: recuento <IdColumn> distinto de en la ventana de <Sequence> tiempo que en transición [prev] --> --> [siguiente]. 
    muestras: una matriz de <IdColumn>identificadores (desde ) correspondientes a la secuencia de la fila (se devuelve un máximo de 128 identificadores). 

* Tabla #3 - secuencia siguiente dcount TimelineColumn: la ventana de tiempo analizada siguiente: el siguiente estado (puede estar vacío si había algún usuario que solo tenía eventos para la secuencia buscada, pero no cualquier evento que la siguiera). 
    dcount: recuento <IdColumn> distinto de en la ventana de <Sequence> tiempo que en transición [prev] --> --> [siguiente]. 
    muestras: una matriz de <IdColumn>identificadores (desde ) correspondientes a la secuencia de la fila (se devuelve un máximo de 128 identificadores). 


**Ejemplos**

### <a name="exploring-storm-events"></a>Explorar eventos de tormenta 

La siguiente consulta examina la tabla StormEvents (estadísticas meteorológicas para 2007) y muestra qué evento ocurre antes/después de que se produjeran todos los eventos Tornado en 2007.

```kusto
// Looking on StormEvents statistics: 
// Q1: What happens before Tornado event?
// Q2: What happens after Tornado event?
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, dynamic(['Tornado']))
```

El resultado incluye 3 tablas:

* Tabla #1: Todas las variantes posibles de lo que sucedió antes y después de la secuencia. Por ejemplo, la segunda línea indica que había 87 eventos diferentes que tenían la siguiente secuencia:`Hail` -> `Tornado` -> `Hail`


|StartTime|prev|Siguiente|dcount|
|---|---|---|---|
|2007-01-01 00:00:00.0000000|||293|
|2007-01-01 00:00:00.0000000|Granizo|Granizo|87|
|2007-01-01 00:00:00.0000000|Viento de tormenta|Viento de tormenta|77|
|2007-01-01 00:00:00.0000000|Granizo|Viento de tormenta|28|
|2007-01-01 00:00:00.0000000|Granizo||28|
|2007-01-01 00:00:00.0000000||Granizo|27|
|2007-01-01 00:00:00.0000000||Viento de tormenta|25|
|2007-01-01 00:00:00.0000000|Viento de tormenta|Granizo|24|
|2007-01-01 00:00:00.0000000|Viento de tormenta||24|
|2007-01-01 00:00:00.0000000|Riada|Riada|12|
|2007-01-01 00:00:00.0000000|Viento de tormenta|Riada|8|
|2007-01-01 00:00:00.0000000|Riada||8|
|2007-01-01 00:00:00.0000000|Nube de embudo|Viento de tormenta|6|
|2007-01-01 00:00:00.0000000||Nube de embudo|6|
|2007-01-01 00:00:00.0000000||Riada|6|
|2007-01-01 00:00:00.0000000|Nube de embudo|Nube de embudo|6|
|2007-01-01 00:00:00.0000000|Granizo|Riada|4|
|2007-01-01 00:00:00.0000000|Riada|Viento de tormenta|4|
|2007-01-01 00:00:00.0000000|Granizo|Nube de embudo|4|
|2007-01-01 00:00:00.0000000|Nube de embudo|Granizo|4|
|2007-01-01 00:00:00.0000000|Nube de embudo||4|
|2007-01-01 00:00:00.0000000|Viento de tormenta|Nube de embudo|3|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|Viento de tormenta|2|
|2007-01-01 00:00:00.0000000|Riada|Nube de embudo|2|
|2007-01-01 00:00:00.0000000|Riada|Granizo|2|
|2007-01-01 00:00:00.0000000|Viento fuerte|Viento de tormenta|1|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|Riada|1|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|Granizo|1|
|2007-01-01 00:00:00.0000000|Granizo|Inundación|1|
|2007-01-01 00:00:00.0000000|Relámpago|Granizo|1|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|Relámpago|1|
|2007-01-01 00:00:00.0000000|Nube de embudo|Lluvia fuerte|1|
|2007-01-01 00:00:00.0000000|Riada|Inundación|1|
|2007-01-01 00:00:00.0000000|Inundación|Riada|1|
|2007-01-01 00:00:00.0000000||Lluvia fuerte|1|
|2007-01-01 00:00:00.0000000|Nube de embudo|Relámpago|1|
|2007-01-01 00:00:00.0000000|Relámpago|Viento de tormenta|1|
|2007-01-01 00:00:00.0000000|Inundación|Viento de tormenta|1|
|2007-01-01 00:00:00.0000000|Granizo|Relámpago|1|
|2007-01-01 00:00:00.0000000||Relámpago|1|
|2007-01-01 00:00:00.0000000|Tormenta tropical|Huracán (Typhoon)|1|
|2007-01-01 00:00:00.0000000|Inundación costera||1|
|2007-01-01 00:00:00.0000000|Corriente de rotura||1|
|2007-01-01 00:00:00.0000000|Nieve pesada||1|
|2007-01-01 00:00:00.0000000|Viento fuerte||1|

* Tabla #2: mostrar todos los eventos distintos agrupados por evento anterior. Por ejemplo, la segunda línea muestra que `Hail` hubo un total de 150 eventos de lo que ocurrió justo antes`Tornado`

|StartTime|prev|Dcount|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||331|
|2007-01-01 00:00:00.0000000|Granizo|150|
|2007-01-01 00:00:00.0000000|Viento de tormenta|135|
|2007-01-01 00:00:00.0000000|Riada|28|
|2007-01-01 00:00:00.0000000|Nube de embudo|22|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|5|
|2007-01-01 00:00:00.0000000|Inundación|2|
|2007-01-01 00:00:00.0000000|Relámpago|2|
|2007-01-01 00:00:00.0000000|Viento fuerte|2|
|2007-01-01 00:00:00.0000000|Nieve pesada|1|
|2007-01-01 00:00:00.0000000|Corriente de rotura|1|
|2007-01-01 00:00:00.0000000|Inundación costera|1|
|2007-01-01 00:00:00.0000000|Tormenta tropical|1|

* Tabla #3: mostrar todos los eventos distintos agrupados por el siguiente evento. Por ejemplo, la segunda línea muestra que `Hail` hubo un total de 143 eventos de que ocurrieron después de`Tornado`

|StartTime|Siguiente|Dcount|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||332|
|2007-01-01 00:00:00.0000000|Granizo|145|
|2007-01-01 00:00:00.0000000|Viento de tormenta|143|
|2007-01-01 00:00:00.0000000|Riada|32|
|2007-01-01 00:00:00.0000000|Nube de embudo|21|
|2007-01-01 00:00:00.0000000|Relámpago|4|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|2|
|2007-01-01 00:00:00.0000000|Inundación|2|
|2007-01-01 00:00:00.0000000|Huracán (Typhoon)|1|

Ahora, vamos a tratar de averiguar cómo continúa la siguiente secuencia:  
`Hail` -> `Tornado` -> `Thunderstorm Wind`

```kusto
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, 
dynamic(['Hail', 'Tornado', 'Thunderstorm Wind']))
```

Saltar `Table #1` y `Table #2`, y `Table #3` mirar en - `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` podemos concluir que la secuencia `Hail` en 92 eventos terminó `Tornado` con esta secuencia, continuó como en 41 eventos, y volvió a 14.

|StartTime|Siguiente|Dcount|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||92|
|2007-01-01 00:00:00.0000000|Granizo|41|
|2007-01-01 00:00:00.0000000|Tornado|14|
|2007-01-01 00:00:00.0000000|Riada|11|
|2007-01-01 00:00:00.0000000|Relámpago|2|
|2007-01-01 00:00:00.0000000|Lluvia fuerte|1|
|2007-01-01 00:00:00.0000000|Inundación|1|
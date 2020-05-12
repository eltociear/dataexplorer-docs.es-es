---
title: 'complemento de funnel_sequence: Azure Explorador de datos'
description: En este artículo se describe funnel_sequence complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c68cac70223b4779b4ca0acf33cd9f66d8c91765
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227407"
---
# <a name="funnel_sequence-plugin"></a>complemento funnel_sequence

Calcula el recuento distintivo de usuarios que han tomado una secuencia de Estados y la distribución de los Estados anterior y siguiente que han provocado o seguido de la secuencia. 

```kusto
T | evaluate funnel_sequence(id, datetime_column, startofday(ago(30d)), startofday(now()), 10m, 1d, state_column, dynamic(['S1', 'S2', 'S3']))
```

**Sintaxis**

*T* `| evaluate` `funnel_sequence(` *IdColumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *MaxSequenceStepWindow*, *Step*, *StateColumn*, *Sequence*`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColum*: la referencia de columna debe estar presente en la expresión de origen.
* *TimelineColumn*: la referencia de columna que representa la escala de tiempo debe estar presente en la expresión de origen.
* *Start*: valor constante escalar del período de inicio del análisis.
* *End*: valor constante escalar del período de finalización del análisis.
* *MaxSequenceStepWindow*: valor constante escalar del intervalo de tiempo máximo permitido entre dos pasos secuenciales de la secuencia.
* *Paso*: valor constante escalar del período de paso de análisis (bin).
* *StateColumn*: la referencia de columna que representa el estado debe estar presente en la expresión de origen.
* *Sequence*: matriz dinámica constante con los valores de secuencia (los valores se buscan en `StateColumn` ).

**Devuelve**

Devuelve tres tablas de salida, que son útiles para crear un diagrama de Sankey para la secuencia analizada:

* Tabla #1-Prev-Sequence-Next `dcount` TimelineColumn: ventana de tiempo analizada Prev: el estado anterior (puede estar vacío si hay usuarios que solo tenían eventos para la secuencia de búsqueda, pero no eventos anteriores). 
    Next: el siguiente estado (puede estar vacío si hay usuarios que solo tenían eventos para la secuencia de búsqueda, pero no los eventos que lo siguieron). 
    `dcount`: recuento distintivo de `IdColumn` en el período de tiempo en el que se ha pasado `prev`  -->  `Sequence`  -->  `next` . 
    ejemplos: una matriz de identificadores (de `IdColumn` ) correspondiente a la secuencia de la fila (se devuelven un máximo de 128 ID.). 

* Tabla #2-Prev-Sequence `dcount` TimelineColumn: la ventana de tiempo analizada Prev: el estado anterior (puede estar vacío si hay usuarios que solo tenían eventos para la secuencia de búsqueda, pero no eventos anteriores). 
    `dcount`: recuento distintivo de `IdColumn` en el período de tiempo en el que se ha pasado `prev`  -->  `Sequence`  -->  `next` . 
    ejemplos: una matriz de identificadores (de `IdColumn` ) correspondiente a la secuencia de la fila (se devuelven un máximo de 128 ID.). 

* Tabla #3-Sequence-Next `dcount` TimelineColumn: la ventana de tiempo analizada Next: Next State (puede estar vacía si hay usuarios que solo tenían eventos para la secuencia buscada, pero no eventos que los siguieron). 
    `dcount`: recuento distintivo de `IdColumn` en el período de tiempo en el que se ha pasado `prev`  -->  `Sequence`  -->  `next` .
    ejemplos: una matriz de identificadores (de `IdColumn` ) correspondiente a la secuencia de la fila (se devuelven un máximo de 128 ID.). 


**Ejemplos**

### <a name="exploring-storm-events"></a>Exploración de eventos de Storm 

La siguiente consulta examina la tabla StormEvents (estadísticas meteorológicas para 2007) y muestra los eventos que se produjeron antes o después de que se produjeran todos los eventos del tornado en 2007.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Looking on StormEvents statistics: 
// Q1: What happens before Tornado event?
// Q2: What happens after Tornado event?
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, dynamic(['Tornado']))
```

El resultado incluye tres tablas:

* Tabla #1: todas las variantes posibles de lo que sucedió antes y después de la secuencia. Por ejemplo, la segunda línea significa que hubo 87 eventos diferentes que tenían la siguiente secuencia:`Hail` -> `Tornado` -> `Hail`


|`StartTime`|`prev`|`next`|`dcount`|
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
|2007-01-01 00:00:00.0000000|Lluvia intensa|Viento de tormenta|2|
|2007-01-01 00:00:00.0000000|Riada|Nube de embudo|2|
|2007-01-01 00:00:00.0000000|Riada|Granizo|2|
|2007-01-01 00:00:00.0000000|Viento fuerte|Viento de tormenta|1|
|2007-01-01 00:00:00.0000000|Lluvia intensa|Riada|1|
|2007-01-01 00:00:00.0000000|Lluvia intensa|Granizo|1|
|2007-01-01 00:00:00.0000000|Granizo|Inundación|1|
|2007-01-01 00:00:00.0000000|Vertiginosa|Granizo|1|
|2007-01-01 00:00:00.0000000|Lluvia intensa|Vertiginosa|1|
|2007-01-01 00:00:00.0000000|Nube de embudo|Lluvia intensa|1|
|2007-01-01 00:00:00.0000000|Riada|Inundación|1|
|2007-01-01 00:00:00.0000000|Inundación|Riada|1|
|2007-01-01 00:00:00.0000000||Lluvia intensa|1|
|2007-01-01 00:00:00.0000000|Nube de embudo|Vertiginosa|1|
|2007-01-01 00:00:00.0000000|Vertiginosa|Viento de tormenta|1|
|2007-01-01 00:00:00.0000000|Inundación|Viento de tormenta|1|
|2007-01-01 00:00:00.0000000|Granizo|Vertiginosa|1|
|2007-01-01 00:00:00.0000000||Vertiginosa|1|
|2007-01-01 00:00:00.0000000|Storm tropicales|Huracán (Typhoon)|1|
|2007-01-01 00:00:00.0000000|Inundación costera||1|
|2007-01-01 00:00:00.0000000|Copiar actual||1|
|2007-01-01 00:00:00.0000000|Nieve pesado||1|
|2007-01-01 00:00:00.0000000|Viento fuerte||1|

* Tabla #2: muestra todos los eventos distintos agrupados por el evento anterior. Por ejemplo, la segunda línea muestra que hubo un total de 150 eventos de `Hail` que ocurrieron justo antes `Tornado` .

|`StartTime`|`prev`|`dcount`|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||331|
|2007-01-01 00:00:00.0000000|Granizo|150|
|2007-01-01 00:00:00.0000000|Viento de tormenta|135|
|2007-01-01 00:00:00.0000000|Riada|28|
|2007-01-01 00:00:00.0000000|Nube de embudo|22|
|2007-01-01 00:00:00.0000000|Lluvia intensa|5|
|2007-01-01 00:00:00.0000000|Inundación|2|
|2007-01-01 00:00:00.0000000|Vertiginosa|2|
|2007-01-01 00:00:00.0000000|Viento fuerte|2|
|2007-01-01 00:00:00.0000000|Nieve pesado|1|
|2007-01-01 00:00:00.0000000|Copiar actual|1|
|2007-01-01 00:00:00.0000000|Inundación costera|1|
|2007-01-01 00:00:00.0000000|Storm tropicales|1|

* Tabla #3: muestra todos los eventos distintos agrupados por evento siguiente. Por ejemplo, la segunda línea muestra que hubo un total de 143 eventos de `Hail` que se produjeron después de `Tornado` .

|`StartTime`|`next`|`dcount`|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||332|
|2007-01-01 00:00:00.0000000|Granizo|145|
|2007-01-01 00:00:00.0000000|Viento de tormenta|143|
|2007-01-01 00:00:00.0000000|Riada|32|
|2007-01-01 00:00:00.0000000|Nube de embudo|21|
|2007-01-01 00:00:00.0000000|Vertiginosa|4|
|2007-01-01 00:00:00.0000000|Lluvia intensa|2|
|2007-01-01 00:00:00.0000000|Inundación|2|
|2007-01-01 00:00:00.0000000|Huracán (Typhoon)|1|

Ahora, vamos a intentar averiguar cómo continúa la siguiente secuencia:  
`Hail` -> `Tornado` -> `Thunderstorm Wind`

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, 
dynamic(['Hail', 'Tornado', 'Thunderstorm Wind']))
```

Omitir `Table #1` y `Table #2` , y examinar `Table #3` , podemos concluir que `Hail`  ->  `Tornado`  ->  `Thunderstorm Wind` la secuencia en 92 eventos finalizaron con esta secuencia, continuar como `Hail` en 41 eventos y volver a `Tornado` en 14.

|`StartTime`|`next`|`dcount`|
|---------|-----|------|
|2007-01-01 00:00:00.0000000||92|
|2007-01-01 00:00:00.0000000|Granizo|41|
|2007-01-01 00:00:00.0000000|Tornado|14|
|2007-01-01 00:00:00.0000000|Riada|11|
|2007-01-01 00:00:00.0000000|Vertiginosa|2|
|2007-01-01 00:00:00.0000000|Lluvia intensa|1|
|2007-01-01 00:00:00.0000000|Inundación|1|

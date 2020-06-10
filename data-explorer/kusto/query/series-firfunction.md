---
title: 'series_fir (): Explorador de datos de Azure'
description: En este artículo se describe series_fir () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: 616fee7b0a1b6852f66d3db22846b2645e03135f
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84665017"
---
# <a name="series_fir"></a>series_fir()

Aplica un filtro de respuesta finita al impulso (FIR) en una serie.  

La función toma una expresión que contiene una matriz numérica dinámica como entrada y aplica un filtro de [respuesta de impulso finito](https://en.wikipedia.org/wiki/Finite_impulse_response) . Al especificar los `filter` coeficientes, se puede usar para calcular una media móvil, suavizado, detección de cambios y muchos otros casos de uso. La función toma como entrada la columna que contiene la matriz dinámica y una matriz dinámica estática de los coeficientes del filtro, y aplica el filtro en la columna. De este modo, genera una columna de matriz dinámica que contiene el resultado filtrado.  

**Sintaxis**

`series_fir(`*x* `,` *filtro* x [ `,` *Normalize*[ `,` *Center*]]`)`

**Argumentos**

* *x*: celda de matriz dinámica de valores numéricos. Normalmente, el resultado de los operadores [Make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md) .
* *Filter*: expresión constante que contiene los coeficientes del filtro (almacenado como una matriz dinámica de valores numéricos).
* *Normalize*: valor booleano opcional que indica si se debe normalizar el filtro. Es decir, dividido por la suma de los coeficientes. Si el filtro contiene valores negativos, *Normalize* debe especificarse como `false` ; de lo contrario, el resultado será `null` . Si no se especifica, se supone un valor predeterminado de *Normalize* , en función de la presencia de valores negativos en el *filtro*. Si *Filter* contiene al menos un valor negativo, se supone que *Normalize* es `false` .  
La normalización es una manera cómoda de asegurarse de que la suma de los coeficientes sea 1. Después, el filtro no amplifica ni atenúa la serie. Por ejemplo, la media móvil de cuatro bins podría especificarse mediante *Filter*= [1, 1, 1, 1] y *normalizado*= true, que es más fácil que escribir [0.25, 0.25.0.25, 0.25].
* *Center*: un valor booleano opcional que indica si el filtro se aplica de forma simétrica en una ventana de tiempo antes y después del punto actual, o en una ventana de tiempo desde el punto actual hacia atrás. De forma predeterminada, Center es false, que se ajusta al escenario de transmisión de datos, donde solo se puede aplicar el filtro en los puntos actual y anterior. Sin embargo, para el procesamiento ad hoc, puede establecerlo en `true` , manteniéndolo sincronizado con la serie temporal de. Vea los ejemplos siguientes. Este parámetro controla el retraso de [Grupo](https://en.wikipedia.org/wiki/Group_delay_and_phase_delay)del filtro.

**Ejemplos**

* Calcule una media móvil de cinco puntos estableciendo *Filter*= [1, 1, 1, 1, 1] y *Normalize* = `true` (valor predeterminado). Tenga en cuenta el efecto de *Center* = `false` (valor predeterminado) frente a `true` :

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range t from bin(now(), 1h)-23h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS', val=dynamic([0,0,0,0,0,0,0,0,0,10,20,40,100,40,20,10,0,0,0,0,0,0,0,0]), t
| extend 5h_MovingAvg=series_fir(val, dynamic([1,1,1,1,1])),
         5h_MovingAvg_centered=series_fir(val, dynamic([1,1,1,1,1]), true, true)
| render timechart
```

Esta consulta devuelve lo siguiente:  
*5h_MovingAvg*: filtro de media móvil de cinco puntos. Se suaviza la punta y su pico se desplaza en (5-1)/2 = 2 h.  
*5h_MovingAvg_centered*: igual, pero estableciendo `center=true` , el pico permanece en su ubicación original.

:::image type="content" source="images/series-firfunction/series-fir.png" alt-text="Serie FIR" border="false":::

* Para calcular la diferencia entre un punto y su anterior, establezca *Filter*= [1,-1].

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range t from bin(now(), 1h)-11h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS',t,value=dynamic([0,0,0,0,2,2,2,2,3,3,3,3])
| extend diff=series_fir(value, dynamic([1,-1]), false, false)
| render timechart
```

:::image type="content" source="images/series-firfunction/series-fir2.png" alt-text="Serie FIR 2" border="false":::

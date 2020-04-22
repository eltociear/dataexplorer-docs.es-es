---
title: series_fir() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fir() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: bbdf93504be431d77c19a881cf79d1e1d3d400ac
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663567"
---
# <a name="series_fir"></a>series_fir()

Aplica un filtro de respuesta de impulso finito en una serie.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica un filtro de respuesta de [impulso finito.](https://en.wikipedia.org/wiki/Finite_impulse_response) Al especificar `filter` los coeficientes, se puede utilizar para calcular la media móvil, suavizado, detección de cambios y muchos más casos de uso. La función toma como entrada la columna que contiene la matriz dinámica y una matriz dinámica estática de los coeficientes del filtro, y aplica el filtro en la columna. De este modo, genera una columna de matriz dinámica que contiene el resultado filtrado.  

**Sintaxis**

`series_fir(`*x* `,` *filter* filtro`,` [`,` *normalizar*[ *centro*]]`)`

**Argumentos**

* *x*: Celda de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list.](makelist-aggfunction.md)
* *filtro*: Una expresión constante que contiene los coeficientes del filtro (almacenado como una matriz dinámica de valores numéricos).
* *normalizar*: Valor booleano opcional que indica si el filtro debe normalizarse (es decir, dividirse por la suma de los coeficientes). Si *el filtro* contiene valores negativos, se debe especificar *normalize* como `false`, de lo contrario el resultado será `null`. Si no se especifica, se asumirá un valor predeterminado de *normalizar* en función de la presencia de valores `false`negativos en el *filtro:* si el *filtro* contiene al menos un valor negativo, se supone que es *normalizar* .  
La normalización es una manera conveniente de asegurarse de que la suma de los coeficientes es 1 y por lo tanto el filtro no amplifica ni atenúa la serie. Por ejemplo, la media móvil de 4 bins podría especificarse mediante *el filtro*[1,1,1,1] y *la normalización*de true, que es más fácil que escribir [0.25,0.25.0.25,0.25,0.25].
* *Centro*: Un valor booleano opcional que indica si el filtro se aplica simétricamente en una ventana de tiempo antes y después del punto actual, o en una ventana de tiempo desde el punto actual hacia atrás. De forma predeterminada, el valor de center es false, que ajusta el escenario de datos de streaming, donde solo podemos aplicar el filtro en los puntos actual y anterior. Sin embargo, para realizar el procesamiento ad hoc, puede establecerlo en true, así se mantendrá sincronizado con la serie temporal (vea los ejemplos siguientes). Técnicamente hablando, este parámetro controla el retardo de [grupo](https://en.wikipedia.org/wiki/Group_delay_and_phase_delay)del filtro.

**Ejemplos**

* El cálculo de una media móvil de 5 puntos se puede realizar estableciendo el *filtro*[1,1,1,1,1] y *normalizando* = `true` (predeterminado). Tenga en *center* = `false` cuenta el efecto `true`de centro (predeterminado) vs.

```kusto
range t from bin(now(), 1h)-23h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS', val=dynamic([0,0,0,0,0,0,0,0,0,10,20,40,100,40,20,10,0,0,0,0,0,0,0,0]), t
| extend 5h_MovingAvg=series_fir(val, dynamic([1,1,1,1,1])),
         5h_MovingAvg_centered=series_fir(val, dynamic([1,1,1,1,1]), true, true)
| render timechart
```

Esta consulta devuelve lo siguiente:  
*5h_MovingAvg*: Filtro de media móvil de 5 puntos. Se suaviza la punta y su pico se desplaza en (5-1)/2 = 2 h.  
*5h_MovingAvg_centered:* lo mismo, pero con la configuración center-true, hace que el pico permanezca en su ubicación original.

:::image type="content" source="images/samples/series-fir.png" alt-text="Abeto de la serie":::

* El cálculo de la diferencia entre un punto y el anterior se puede realizar estableciendo el *filtro*.[1,-1]:

```kusto
range t from bin(now(), 1h)-11h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS',t,value=dynamic([0,0,0,0,2,2,2,2,3,3,3,3])
| extend diff=series_fir(value, dynamic([1,-1]), false, false)
| render timechart
```
:::image type="content" source="images/samples/series-fir2.png" alt-text="Abeto de la serie 2":::

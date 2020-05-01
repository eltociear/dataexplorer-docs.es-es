---
title: series_fir ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe series_fir () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: 4641da6481365d13919de59708387b689581c9ce
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618820"
---
# <a name="series_fir"></a>series_fir()

Aplica un filtro de respuesta finita al impulso en una serie.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica un filtro de [respuesta de impulso finito](https://en.wikipedia.org/wiki/Finite_impulse_response) . Al especificar los coeficientes, se puede usar para calcular la `filter` media móvil, el suavizado, la detección de cambios y muchos otros casos de uso. La función toma como entrada la columna que contiene la matriz dinámica y una matriz dinámica estática de los coeficientes del filtro, y aplica el filtro en la columna. De este modo, genera una columna de matriz dinámica que contiene el resultado filtrado.  

**Sintaxis**

`series_fir(`*x*`,` *filtro* x [`,` *Normalize*[`,` *Center*]]`)`

**Argumentos**

* *x*: celda de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de operadores de [creación de serie](make-seriesoperator.md) o de [make_list](makelist-aggfunction.md) .
* *Filter*: expresión constante que contiene los coeficientes del filtro (almacenado como una matriz dinámica de valores numéricos).
* *Normalize*: valor booleano opcional que indica si el filtro debe normalizarse (es decir, dividido por la suma de los coeficientes). Si el *filtro* contiene valores negativos, *Normalize* debe especificarse `false`como; de lo contrario `null`, el resultado será. Si no se especifica, se asumirá un valor predeterminado de *Normalize* en función de la presencia de valores negativos en el *filtro*: Si *Filter* contiene al menos un valor negativo, se supone que `false` *Normalize* es.  
La normalización es una manera cómoda de asegurarse de que la suma de los coeficientes es 1 y, por lo tanto, el filtro no amplifica o atenúa la serie. Por ejemplo, la media móvil de 4 depósitos podría especificarse mediante *Filter*= [1, 1, 1, 1] y *normalizado*= true, que es más fácil que escribir [0,25, 0.25.0.25, 0,25].
* *Center*: un valor booleano opcional que indica si el filtro se aplica de forma simétrica en una ventana de tiempo antes y después del punto actual, o en una ventana de tiempo desde el punto actual hacia atrás. De forma predeterminada, el valor de center es false, que ajusta el escenario de datos de streaming, donde solo podemos aplicar el filtro en los puntos actual y anterior. Sin embargo, para realizar el procesamiento ad hoc, puede establecerlo en true, así se mantendrá sincronizado con la serie temporal (vea los ejemplos siguientes). Técnicamente hablando, este parámetro controla el retraso de [Grupo](https://en.wikipedia.org/wiki/Group_delay_and_phase_delay)del filtro.

**Ejemplos**

* El cálculo de una media móvil de 5 puntos puede realizarse estableciendo el *filtro*= [1, 1, 1, 1, 1] y *normalizar* = `true` (valor predeterminado). Tenga en cuenta el efecto de *Center* = `false` (valor predeterminado `true`) frente a:

```kusto
range t from bin(now(), 1h)-23h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS', val=dynamic([0,0,0,0,0,0,0,0,0,10,20,40,100,40,20,10,0,0,0,0,0,0,0,0]), t
| extend 5h_MovingAvg=series_fir(val, dynamic([1,1,1,1,1])),
         5h_MovingAvg_centered=series_fir(val, dynamic([1,1,1,1,1]), true, true)
| render timechart
```

Esta consulta devuelve lo siguiente:  
*5h_MovingAvg*: filtro de media móvil de 5 puntos. Se suaviza la punta y su pico se desplaza en (5-1)/2 = 2 h.  
*5h_MovingAvg_centered*: igual pero con el centro de configuración = true, hace que el pico permanezca en su ubicación original.

:::image type="content" source="images/series-firfunction/series-fir.png" alt-text="Serie FIR" border="false":::

* El cálculo de la diferencia entre un punto y su anterior se puede realizar mediante el establecimiento de *Filter*= [1,-1]:

```kusto
range t from bin(now(), 1h)-11h to bin(now(), 1h) step 1h
| summarize t=make_list(t)
| project id='TS',t,value=dynamic([0,0,0,0,2,2,2,2,3,3,3,3])
| extend diff=series_fir(value, dynamic([1,-1]), false, false)
| render timechart
```

:::image type="content" source="images/series-firfunction/series-fir2.png" alt-text="Serie FIR 2" border="false":::

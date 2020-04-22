---
title: series_decompose_forecast() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_decompose_forecast() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: 67f464949bbc432e73932c4d8bff8290ef8f0f8f
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663547"
---
# <a name="series_decompose_forecast"></a>series_decompose_forecast()

Pronóstico basado en la descomposición de la serie.

Toma una expresión que contiene una serie (matriz numérica dinámica) como entrada y predice los valores de los últimos puntos finales (consulte [series_decompose](series-decomposefunction.md) para obtener más detalles sobre el método de descomposición).
 
**Sintaxis**

`series_decompose_forecast(`Tendencia de *temporada* `,` *Trend* `,` `,` de *puntos* `[,` *de serie* *Seasonality_threshold*`])`

**Argumentos**

* *Serie*: Celda de matriz dinámica que es una matriz de valores numéricos. Normalmente, la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list.](makelist-aggfunction.md)
* *Puntos*: Entero que especifica el número de puntos al final de la serie para predecir (previsión). Estos puntos se excluyen del proceso de aprendizaje (regresión).
* *Estacionalidad*: Un entero que controla el análisis estacional, que contiene uno de los siguientes:
    * -1: detectar automáticamente la estacionalidad mediante [series_periods_detect](series-periods-detectfunction.md) (predeterminado). 
    * período: entero positivo, especificando el período esperado en número de ubicaciones. Por ejemplo, si la serie está en contenedores de 1h, un período semanal es de 168 ubicaciones.
    * 0: sin estacionalidad (omita la extracción de este componente).   
* *Tendencia*: Una cadena que controla el análisis de tendencia, que contiene uno de los siguientes:
    * "linefit": extrae el componente de tendencia utilizando la regresión lineal (predeterminado).    
    * "avg": define el componente de tendencia como average(x).
    * "none": no hay tendencia, omita la extracción de este componente.   
* *Seasonality_threshold*: El umbral para la puntuación de estacionalidad cuando *Seasonality* se establece en auto-detect, el umbral de puntuación predeterminado es `0.6`. Para obtener más información, consulte [series_periods_detect](series-periods-detectfunction.md).

**devolución**

 Una matriz dinámica con la serie pronosticada
  

**Notas**

* La matriz dinámica de la serie de entrada original debe incluir un número de *ranuras* de puntos que se pronosticarán, esto se suele hacer mediante el uso [de make-series](make-seriesoperator.md) y especificando la hora de finalización en el rango que incluye el período de tiempo que se va a pronosticar.
    
* Ya sea la estacionalidad y / o tendencia debe estar habilitado, de lo contrario la función es redundante y sólo devuelve una serie llena de ceros.

**Ejemplo**

En el siguiente ejemplo generamos una serie de 4 semanas en un grano por `make-series` hora con estacionalidad semanal y una pequeña tendencia al alza, luego usamos y agregamos otra semana vacía a la serie. `series_decompose_forecast`se llama con una semana (24*7 puntos), detecta automáticamente la estacionalidad y la tendencia y genera un pronóstico de todo el período de 5 semanas. 

```kusto
let ts=range t from 1 to 24*7*4 step 1 // generate 4 weeks of hourly data
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| make-series y=max(y) on Timestamp in range(datetime(2018-03-01 05:00), datetime(2018-03-01 05:00)+24*7*5h, 1h); // create a time series of 5 weeks (last week is empty)
ts 
| extend y_forcasted = series_decompose_forecast(y, 24*7)  // forecast a week forward
| render timechart 
```

:::image type="content" source="images/samples/series-decompose-forecast.png" alt-text="Pronóstico de descomponeción de series":::

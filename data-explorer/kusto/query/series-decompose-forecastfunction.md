---
title: 'series_decompose_forecast (): Explorador de datos de Azure'
description: En este artículo se describe series_decompose_forecast () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: 7aafc6ce041395a914787ed2b406d88aa9910238
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717173"
---
# <a name="series_decompose_forecast"></a>series_decompose_forecast()

Previsión basada en la descomposición de la serie.

Toma una expresión que contiene una serie (matriz numérica dinámica) como entrada y predice los valores de los últimos puntos finales. Para obtener más información, vea [series_decompose](series-decomposefunction.md).
 
**Sintaxis**

`series_decompose_forecast(`*Serie* `,` de *Puntos* `[,` de *Estacionalidad* `,` *Tendencia* `,` *Seasonality_threshold*`])`

**Argumentos**

* *Series*: celda de matriz dinámica de valores numéricos. Normalmente, el resultado de los operadores [Make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md) .
* *Points*: entero que especifica el número de puntos al final de la serie que se va a predecir (previsión). Estos puntos se excluyen del proceso de aprendizaje (regresión).
* *Estacionalidad*: un entero que controla el análisis estacional, que contiene uno de los siguientes:
    * -1: detección automática de estacionalidad mediante [series_periods_detect](series-periods-detectfunction.md) (valor predeterminado).
    * Period: entero positivo, que especifica el período esperado en número de ubicaciones. Por ejemplo, si la serie está en las bandejas 1H, un período semanal es 168 bandejas.
    * 0: sin estacionalidad (se omite la extracción de este componente).
* *Tendencia*: una cadena que controla el análisis de tendencias, que contiene uno de los siguientes:
    * `linefit`: Extrae el componente de tendencia mediante la regresión lineal (valor predeterminado).
    * `avg`: Define el componente de tendencia como promedio (x).
    * `none`: Sin tendencia, omitir la extracción de este componente.
* *Seasonality_threshold*: el umbral de puntuación de estacionalidad cuando la *estacionalidad* está establecida en detección automática. El umbral de puntuación predeterminado es `0.6` . Para obtener más información, vea [series_periods_detect](series-periods-detectfunction.md).

**Devolver**

 Matriz dinámica con la serie pronosticada.

> [!NOTE]
> * La matriz dinámica de la serie de entrada original debe incluir un número de ranuras de *puntos* que se van a predecir. La previsión se realiza normalmente mediante la [serie de creación](make-seriesoperator.md) y especifica la hora de finalización en el intervalo que incluye el período de tiempo para la previsión.
> * Se debe habilitar la estacionalidad o tendencia, de lo contrario la función es redundante y simplemente devuelve una serie rellena con ceros.

**Ejemplo**

En el ejemplo siguiente, se genera una serie de cuatro semanas en un grano por hora, con estacionalidad semanal y una pequeña tendencia ascendente. A continuación, usamos `make-series` y agregamos otra semana vacía a la serie. `series_decompose_forecast`se llama con una semana (24 * 7 puntos) y detecta automáticamente la estacionalidad y la tendencia, y genera una previsión de todo el período de cinco semanas.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

:::image type="content" source="images/series-decompose-forecastfunction/series-decompose-forecast.png" alt-text="Previsión de descomposición de series":::
 

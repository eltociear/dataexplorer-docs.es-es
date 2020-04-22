---
title: series_fit_2lines() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe series_fit_2lines() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 05163735c88c3229c13d76e3f8fde9bff091b239
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663501"
---
# <a name="series_fit_2lines"></a>series_fit_2lines()

Aplica dos segmentos de regresión lineal en una serie, devolviendo varias columnas.  

Toma una expresión que contiene la matriz numérica dinámica como entrada y aplica dos segmentos de [regresión lineal](https://en.wikipedia.org/wiki/Segmented_regression) para identificar y cuantificar el cambio de tendencia en una serie. La función itera en los índices de la serie y en cada iteración divide la serie en 2 partes, se ajusta a una línea separada (usando [series_fit_line()](series-fit-linefunction.md)) a cada parte y calcula el r-cuadrado total. La división recomendada es la que maximizó el R cuadrado; la función devuelve sus parámetros:
* `rsquare`: [r-cuadrado](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. Se trata de un número en el intervalo [0-1], donde 1 es el mejor ajuste posibles, y 0 significa que los datos están totalmente desordenados y que no se ajustan a ninguna línea.
* `split_idx`: el índice de punto de rotura a 2 segmentos (basado en cero)
* `variance`: varianza de los datos de entrada
* `rvariance`: varianza residual que es la varianza entre los valores de datos de entrada los aproximados (por los 2 segmentos de línea).
* `line_fit`: matriz numérica que contiene una serie de valores de la línea mejor ajustada. La longitud de la serie es igual que la de la matriz de entrada. Se usa principalmente para los gráficos.
* `right_rsquare`: r-cuadrado de la línea en el lado derecho de la división, véase [series_fit_line()](series-fit-linefunction.md)
* `right_slope`: pendiente de la línea aproximada derecha (esto es un de y-ax+b)
* `right_interception`: intercepción de la línea izquierda aproximada (esto es b de y-ax+b)
* `right_variance`: varianza de los datos de entrada en el lado derecho de la división
* `right_rvariance`: varianza residual de los datos de entrada en el lado derecho de la división
* `left_rsquare`: r-cuadrado de la línea en el lado izquierdo de la división, véase [series_fit_line()](series-fit-linefunction.md)
* `left_slope`: pendiente de la línea aproximada izquierda (esto es a partir de y-ax+b)
* `left_interception`: intercepción de la línea izquierda aproximada (esto es b de y-ax+b)
* `left_variance`: varianza de los datos de entrada en el lado izquierdo de la división
* `left_rvariance`: varianza residual de los datos de entrada en el lado izquierdo de la división

*Tenga en cuenta* que esta función devuelve varias columnas, por lo tanto, no se puede utilizar como argumento para otra función.

**Sintaxis**

proyecto `series_fit_2lines(` *x*`)`
* Devolverá todas las columnas mencionadas anteriormente con los siguientes nombres: series_fit_2lines_x_rsquare, series_fit_2lines_x_split_idx y etc.
proyecto (rs, si,`series_fit_2lines(`v)*x*`)`
* Devolverá las siguientes columnas: rs (r-cuadrado), si (índice dividido), v (varianza) y el resto se verá como`series_fit_2lines(`series_fit_2lines_x_rvariance, series_fit_2lines_x_line_fit y etc. extender (rs, si, v)*x*`)`
* Devolverá solo: rs (R cuadrado), si (índice de división) y v (varianza).
  
**Argumentos**

* *x*: Matriz dinámica de valores numéricos.  

> [!TIP]
> La forma más conveniente de utilizar esta función es aplicarla a los resultados del operador [make-series.](make-seriesoperator.md)

**Ejemplos**

```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend (Slope,Interception,RSquare,Variance,RVariance,LineFit)=series_fit_line(y), (RSquare2, SplitIdx, Variance2,RVariance2,LineFit2)=series_fit_2lines(y)
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/samples/series-fit-2lines.png" alt-text="Ajuste en serie 2 líneas":::

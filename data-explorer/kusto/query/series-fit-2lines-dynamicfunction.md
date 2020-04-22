---
title: series_fit_2lines_dynamic() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fit_2lines_dynamic() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: efb2743789c363a33e21ba472a945087b9b277cf
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663517"
---
# <a name="series_fit_2lines_dynamic"></a>series_fit_2lines_dynamic()

Aplica dos segmentos de regresión lineal en una serie, devolviendo el objeto dinámico.  

Toma una expresión que contiene la matriz numérica dinámica como entrada y aplica dos segmentos de [regresión lineal](https://en.wikipedia.org/wiki/Segmented_regression) para identificar y cuantificar el cambio de tendencia en una serie. La función itera en los índices de la serie y en cada iteración divide la serie en 2 partes, se ajusta a una línea separada (usando [series_fit_line()](series-fit-linefunction.md) o [series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)) a cada parte y calcula el r-cuadrado total. La mejor división es la que maximizó r-cuadrado; la función devuelve sus parámetros en valor dinámico con el siguiente contenido:
* `rsquare`: [r-cuadrado](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. Se trata de un número en el intervalo [0-1], donde 1 es el mejor ajuste posibles, y 0 significa que los datos están totalmente desordenados y que no se ajustan a ninguna línea.
* `split_idx`: el índice de punto de rotura a 2 segmentos (basado en cero)
* `variance`: varianza de los datos de entrada
* `rvariance`: varianza residual que es la varianza entre los valores de datos de entrada los aproximados (por los 2 segmentos de línea).
* `line_fit`: matriz numérica que contiene una serie de valores de la línea mejor ajustada. La longitud de la serie es igual que la de la matriz de entrada. Se usa principalmente para los gráficos.
* `right.rsquare`: r-cuadrado de la línea en el lado derecho de la división, véase [series_fit_line()](series-fit-linefunction.md) o[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)
* `right.slope`: pendiente de la línea aproximada derecha (esto es un de y-ax+b)
* `right.interception`: intercepción de la línea izquierda aproximada (esto es b de y-ax+b)
* `right.variance`: varianza de los datos de entrada en el lado derecho de la división
* `right.rvariance`: varianza residual de los datos de entrada en el lado derecho de la división
* `left.rsquare`: r-cuadrado de la línea en el lado izquierdo de la división, véase [series_fit_line()](series-fit-linefunction.md) o [series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)
* `left.slope`: pendiente de la línea aproximada izquierda (esto es a partir de y-ax+b)
* `left.interception`: intercepción de la línea izquierda aproximada (esto es b de y-ax+b)
* `left.variance`: varianza de los datos de entrada en el lado izquierdo de la división
* `left.rvariance`: varianza residual de los datos de entrada en el lado izquierdo de la división

Este operador es [series_fit_2lines](series-fit-2linesfunction.md)similar a `series-fit-2lines` series_fit_2lines , pero a diferencia de él devuelve una bolsa dinámica.

**Sintaxis**

`series_fit_2lines_dynamic(`*X*`)`

**Argumentos**

* *x*: Matriz dinámica de valores numéricos.  

> [!TIP]
> La forma más conveniente de utilizar esta función es aplicarla a los resultados del operador [make-series.](make-seriesoperator.md)

**Ejemplos**

```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend LineFit=series_fit_line_dynamic(y).line_fit, LineFit2=series_fit_2lines_dynamic(y).line_fit
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/samples/series-fit-2lines.png" alt-text="Ajuste en serie 2 líneas":::

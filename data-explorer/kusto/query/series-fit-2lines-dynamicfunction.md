---
title: 'series_fit_2lines_dynamic (): Explorador de datos de Azure'
description: En este artículo se describe series_fit_2lines_dynamic () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 6f1a6e4a80bfbc02f9e6f552ceca2ba1bb54eb08
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372729"
---
# <a name="series_fit_2lines_dynamic"></a>series_fit_2lines_dynamic()

Aplica dos regresiones lineales de segmentos en una serie, devolviendo un objeto dinámico.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica [dos regresiones lineales de segmentos](https://en.wikipedia.org/wiki/Segmented_regression) para identificar y cuantificar los cambios de tendencia en una serie. La función recorre en iteración los índices de la serie. En cada iteración, divide la serie en dos partes y ajusta una línea independiente mediante [series_fit_line ()](series-fit-linefunction.md) o [series_fit_line_dynamic ()](series-fit-line-dynamicfunction.md). La función ajusta las líneas a cada una de las dos partes y calcula el valor total de R cuadrado. La mejor división es la que maximiza el cuadrado de R. La función devuelve sus parámetros en valor dinámico con el siguiente contenido:

* `rsquare`: [R-Square](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. Es un número en el intervalo de [0-1], donde 1 es el mejor ajuste posible y 0 significa que los datos están desordenados y no se ajustan a ninguna línea.
* `split_idx`: el índice de punto de interrupción en dos segmentos (basado en cero).
* `variance`: varianza de los datos de entrada.
* `rvariance`: varianza residual que es la varianza entre los valores de los datos de entrada y los aproximados (por los dos segmentos de línea).
* `line_fit`: matriz numérica que contiene una serie de valores de la mejor línea ajustada. La longitud de la serie es igual que la de la matriz de entrada. Se utiliza para los gráficos.
* `right.rsquare`: r: cuadrado de la línea en el lado derecho de la división, vea [series_fit_line ()](series-fit-linefunction.md) o [series_fit_line_dynamic ()](series-fit-line-dynamicfunction.md).
* `right.slope`: pendiente de la línea derecha aproximada (de la forma y = AX + b).
* `right.interception`: interceptación de la línea izquierda aproximada (b de y = AX + b).
* `right.variance`: varianza de los datos de entrada en el lado derecho de la división.
* `right.rvariance`: varianza residual de los datos de entrada en el lado derecho de la división.
* `left.rsquare`: r: cuadrado de la línea en el lado izquierdo de la división; vea [series_fit_line ()]. (series-fit-linefunction.md) o [series_fit_line_dynamic ()](series-fit-line-dynamicfunction.md).
* `left.slope`: pendiente de la línea izquierda aproximada (de la forma y = AX + b).
* `left.interception`: interceptación de la línea izquierda aproximada (de la forma y = AX + b).
* `left.variance`: varianza de los datos de entrada en el lado izquierdo de la división.
* `left.rvariance`: varianza residual de los datos de entrada en el lado izquierdo de la división.

Este operador es similar a [series_fit_2lines](series-fit-2linesfunction.md). A diferencia `series-fit-2lines` de, devuelve una bolsa dinámica.

**Sintaxis**

`series_fit_2lines_dynamic(`*x1*`)`

**Argumentos**

* *x*: matriz dinámica de valores numéricos.  

> [!TIP]
> La manera más cómoda de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend LineFit=series_fit_line_dynamic(y).line_fit, LineFit2=series_fit_2lines_dynamic(y).line_fit
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/series-fit-2lines/series-fit-2lines.png" alt-text="Serie de 2 líneas de ajuste":::

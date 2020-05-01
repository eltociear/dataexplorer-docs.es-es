---
title: series_fit_2lines ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe series_fit_2lines () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 6c16b535962271a7aaf4acad63f52da028b49e33
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618755"
---
# <a name="series_fit_2lines"></a>series_fit_2lines()

Aplica dos regresiones lineales de segmentos en una serie y devuelve varias columnas.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica [dos regresiones lineales de segmentos](https://en.wikipedia.org/wiki/Segmented_regression) para identificar y cuantificar el cambio de tendencia en una serie. La función recorre en iteración los índices de la serie y en cada iteración divide la serie en dos partes, ajusta una línea independiente (usando [series_fit_line ()](series-fit-linefunction.md)) a cada parte y calcula el total de r-cuadrado. La división recomendada es la que maximizó el R cuadrado; la función devuelve sus parámetros:
* `rsquare`: [r-Square](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. Se trata de un número en el intervalo [0-1], donde 1 es el mejor ajuste posibles, y 0 significa que los datos están totalmente desordenados y que no se ajustan a ninguna línea.
* `split_idx`: el índice de punto de interrupción en 2 segmentos (basado en cero)
* `variance`: varianza de los datos de entrada
* `rvariance`: varianza residual, que es la varianza entre los valores de los datos de entrada y los aproximados (por los dos segmentos de línea).
* `line_fit`: matriz numérica que contiene una serie de valores de la mejor línea ajustada. La longitud de la serie es igual que la de la matriz de entrada. Se usa principalmente para los gráficos.
* `right_rsquare`: r: cuadrado de la línea en el lado derecho de la división, vea [series_fit_line ()](series-fit-linefunction.md)
* `right_slope`: pendiente de la línea derecha aproximada (es decir, de y = AX + b)
* `right_interception`: interceptación de la línea izquierda aproximada (es decir, b de y = AX + b)
* `right_variance`: varianza de los datos de entrada en el lado derecho de la división.
* `right_rvariance`: varianza residual de los datos de entrada en el lado derecho de la división.
* `left_rsquare`: r: cuadrado de la línea en el lado izquierdo de la división, vea [series_fit_line ()](series-fit-linefunction.md)
* `left_slope`: pendiente de la línea izquierda aproximada (es decir, de y = AX + b)
* `left_interception`: interceptación de la línea izquierda aproximada (es decir, b de y = AX + b)
* `left_variance`: varianza de los datos de entrada en el lado izquierdo de la división.
* `left_rvariance`: varianza residual de los datos de entrada en el lado izquierdo de la división.

*Tenga en cuenta* que esta función devuelve varias columnas, por lo que no se puede usar como argumento para otra función.

**Sintaxis**

proyecto `series_fit_2lines(` *x*`)`
* Devolverá todas las columnas mencionadas anteriormente con los nombres siguientes: series_fit_2lines_x_rsquare, series_fit_2lines_x_split_idx, etc.
proyecto (RS, si, v) =`series_fit_2lines(`*x*`)`
* Devolverá las siguientes columnas: RS (r-Square), si (Split index), v (varianza) y el resto tendrá el siguiente aspecto series_fit_2lines_x_rvariance, series_fit_2lines_x_line_fit y etc. Extend (RS, si, v`series_fit_2lines(`) =*x*`)`
* Devolverá solo: rs (R cuadrado), si (índice de división) y v (varianza).
  
**Argumentos**

* *x*: matriz dinámica de valores numéricos.  

> [!TIP]
> La manera más cómoda de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplos**

```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend (Slope,Interception,RSquare,Variance,RVariance,LineFit)=series_fit_line(y), (RSquare2, SplitIdx, Variance2,RVariance2,LineFit2)=series_fit_2lines(y)
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/series-fit-2lines/series-fit-2lines.png" alt-text="Serie de 2 líneas de ajuste":::

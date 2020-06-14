---
title: 'series_fit_2lines (): Explorador de datos de Azure'
description: En este artículo se describe series_fit_2lines () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: a364361ee5e5e260436486db24f1b61e2c21cbc9
ms.sourcegitcommit: 9fc3d8b396dddd2e1d9912845ba7bcc8e31c0267
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/12/2020
ms.locfileid: "84720917"
---
# <a name="series_fit_2lines"></a>series_fit_2lines()

Aplica dos regresiones lineales de segmentos en una serie y devuelve varias columnas.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica [dos regresiones lineales de segmentos](https://en.wikipedia.org/wiki/Segmented_regression) para identificar y cuantificar un cambio de tendencia en una serie. La función recorre en iteración los índices de la serie. En cada iteración, la función divide la serie en dos partes, ajusta una línea independiente (usando [series_fit_line ()](series-fit-linefunction.md)) a cada parte y calcula el total de r-Square. La división recomendada es la que maximizó el R cuadrado; la función devuelve sus parámetros:


|Parámetro  |Descripción  |
|---------|---------|
|`rsquare`     | [R-Square](https://en.wikipedia.org/wiki/Coefficient_of_determination) es la medida estándar de la calidad de ajuste. Es un número en el intervalo [0-1], donde 1 es el mejor ajuste posible y 0 significa que los datos están desordenados y no se ajustan a ninguna línea.        |
|`split_idx`     |   Índice de punto de interrupción en dos segmentos (basado en cero).      |
|`variance`     | Varianza de los datos de entrada.        |
|`rvariance`     | Varianza residual, que es la varianza entre los valores de los datos de entrada y los aproximados (por los dos segmentos de línea).        |
|`line_fit`     | Matriz numérica que contiene una serie de valores de la mejor línea contratada. La longitud de la serie es igual que la de la matriz de entrada. Se usa principalmente para los gráficos.        |
|`right_rsquare`     | R: cuadrado de la línea del lado derecho de la división, vea [series_fit_line ()](series-fit-linefunction.md).        |
|`right_slope`     | Pendiente de la línea derecha aproximada (de la forma y = AX + b).         |
|`right_interception`     |  Interceptación de la línea izquierda aproximada (b de y = AX + b).       |
|`right_variance`    | Varianza de los datos de entrada en el lado derecho de la división.        |
|`right_rvariance`     | Varianza residual de los datos de entrada en el lado derecho de la división.        |
|`left_rsquare`     | R: cuadrado de la línea en el lado izquierdo de la división, vea [series_fit_line ()](series-fit-linefunction.md).        |
|`left_slope`    | Pendiente de la línea izquierda aproximada (de la forma y = AX + b).        |
|`left_interception`     |   Interceptación de la línea izquierda aproximada (de la forma y = AX + b).      |
|`left_variance`     | Varianza de los datos de entrada en el lado izquierdo de la división.        |
|`left_rvariance`     | Varianza residual de los datos de entrada en el lado izquierdo de la división.        |


> [!Note]
> Esta función devuelve varias columnas, por lo que no se puede usar como argumento para otra función.

**Sintaxis**

proyecto `series_fit_2lines(` *x*`)`
* Devolverá todas las columnas mencionadas anteriormente con los nombres siguientes: series_fit_2lines_x_rsquare, series_fit_2lines_x_split_idx etc.

proyecto (RS, si, v) = `series_fit_2lines(` *x*`)`
* Devolverá las siguientes columnas: RS (r-Square), si (Split index), v (varianza) y el resto tendrá el siguiente aspecto series_fit_2lines_x_rvariance, series_fit_2lines_x_line_fit, etc.

Extend (RS, si, v) = `series_fit_2lines(` *x*`)`
* Devolverá solo: rs (R cuadrado), si (índice de división) y v (varianza).
  
**Argumentos**

* *x*: matriz dinámica de valores numéricos.  

> [!TIP]
> La manera más cómoda de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([1,2.2, 2.5, 4.7, 5.0, 12, 10.3, 10.3, 9, 8.3, 6.2])
| extend (Slope,Interception,RSquare,Variance,RVariance,LineFit)=series_fit_line(y), (RSquare2, SplitIdx, Variance2,RVariance2,LineFit2)=series_fit_2lines(y)
| project id, x, y, LineFit, LineFit2
| render timechart
```

:::image type="content" source="images/series-fit-2lines/series-fit-2lines.png" alt-text="Serie de 2 líneas de ajuste":::

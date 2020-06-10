---
title: 'series_fit_line_dynamic (): Explorador de datos de Azure'
description: En este artículo se describe series_fit_line_dynamic () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: f3fb8361cfb281ad39dee7a15a690c2b94c79bea
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84664932"
---
# <a name="series_fit_line_dynamic"></a>series_fit_line_dynamic()

Aplica la regresión lineal en una serie y devuelve un objeto dinámico.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y realiza una [regresión lineal](https://en.wikipedia.org/wiki/Line_fitting) para encontrar la línea que mejor se adapta a ella. Asimismo, debe usarse en las matrices de series temporales, de modo que se ajuste el resultado del operador make-series. Genera un valor dinámico con el siguiente contenido:
* `rsquare`: [r-Square](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. Es un número en el intervalo [0-1], donde 1 es el mejor ajuste posible y 0 significa que los datos están desordenados y no se ajustan a ninguna línea.
* `slope`: Pendiente de la línea aproximada *(el valor a de* *y = AX + b*)
* `variance`: Varianza de los datos de entrada
* `rvariance`: Varianza residual que es la varianza entre los valores de los datos de entrada y los aproximados.
* `interception`: Interceptación de la línea aproximada (el valor *b*de *y = AX + b*)
* `line_fit`: Matriz numérica que contiene una serie de valores de la línea de ajuste óptima. La longitud de la serie es igual que la de la matriz de entrada. Se usa principalmente para los gráficos.

Este operador es similar a [series_fit_line](series-fit-linefunction.md), pero a diferencia `series-fit-line` de que devuelve una bolsa dinámica.

**Sintaxis**

`series_fit_line_dynamic(`*x1*`)`

**Argumentos**

* *x*: matriz dinámica de valores numéricos.

> [!TIP]
> La manera más cómoda de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([2,5,6,8,11,15,17,18,25,26,30,30])
| extend fit=series_fit_line_dynamic(y)
| extend RSquare=fit.rsquare, Slope=fit.slope, Variance=fit.variance,RVariance=fit.rvariance,Interception=fit.interception,LineFit=fit.line_fit
| render timechart
```
 
:::image type="content" source="images/series-fit-line/series-fit-line.png" alt-text="Línea de ajuste de serie":::

| RSquare | Slope | Variance | RVariance | Interception | LineFit                                                                                     |
|---------|-------|----------|-----------|--------------|---------------------------------------------------------------------------------------------|
| 0.982   | 2.730 | 98.628   | 1.686     | -1.666       | 1,064, 3,7945, 6,526, 9,256, 11,987, 14,718, 17,449, 20,180, 22,910, 25,641, 28,371, 31,102 |
 

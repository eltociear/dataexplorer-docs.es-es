---
title: series_fit_line_dynamic() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fit_line_dynamic() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 03bbb143504d0540156d5f18d4ed527ab5d13d38
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663552"
---
# <a name="series_fit_line_dynamic"></a>series_fit_line_dynamic()

Aplica la regresión lineal en una serie, devolviendo el objeto dinámico.  

Toma una expresión que contiene la matriz numérica dinámica como entrada y realiza la [regresión lineal](https://en.wikipedia.org/wiki/Line_fitting) para encontrar la línea que mejor se adapte a ella. Asimismo, debe usarse en las matrices de series temporales, de modo que se ajuste el resultado del operador make-series. Genera un valor dinámico con el siguiente contenido:
* `rsquare`: [r-cuadrado](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. Se trata de un número en el intervalo [0-1], donde 1 es el mejor ajuste posibles, y 0 significa que los datos están totalmente desordenados y que no se ajustan a ninguna línea. 
* `slope`: pendiente de la línea aproximada (esto es a partir de y-ax+b)
* `variance`: varianza de los datos de entrada
* `rvariance`: varianza residual que es la varianza entre los valores de datos de entrada los aproximados.
* `interception`: intercepción de la línea aproximada (esto es b de y-ax+b)
* `line_fit`: matriz numérica que contiene una serie de valores de la línea mejor ajustada. La longitud de la serie es igual que la de la matriz de entrada. Se usa principalmente para los gráficos.

Este operador es [series_fit_line](series-fit-linefunction.md)similar a `series-fit-line` series_fit_line , pero a diferencia de él devuelve una bolsa dinámica.

**Sintaxis**

`series_fit_line_dynamic(`*X*`)`

**Argumentos**

* *x*: Matriz dinámica de valores numéricos.

> [!TIP]
> La forma más conveniente de utilizar esta función es aplicarla a los resultados del operador [make-series.](make-seriesoperator.md)

**Ejemplos**

```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([2,5,6,8,11,15,17,18,25,26,30,30])
| extend fit=series_fit_line_dynamic(y)
| extend RSquare=fit.rsquare, Slope=fit.slope, Variance=fit.variance,RVariance=fit.rvariance,Interception=fit.interception,LineFit=fit.line_fit
| render timechart
```

:::image type="content" source="images/samples/series-fit-line.png" alt-text="Línea de ajuste de serie":::

| RSquare | Slope | Variance | RVariance | Interception | LineFit                                                                                     |
|---------|-------|----------|-----------|--------------|---------------------------------------------------------------------------------------------|
| 0.982   | 2.730 | 98.628   | 1.686     | -1.666       | 1.064, 3.7945, 6.526, 9.256, 11.987, 14.718, 17.449, 20.180, 22.910, 25.641, 28.371, 31.102 |
---
title: 'series_fit_line (): Explorador de datos de Azure'
description: En este artículo se describe series_fit_line () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: f0401a5b10d2feb74c629e6b04b127e6d36057ad
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717183"
---
# <a name="series_fit_line"></a>series_fit_line()

Aplica la regresión lineal en una serie y devuelve varias columnas.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y realiza una [regresión lineal](https://en.wikipedia.org/wiki/Line_fitting) para encontrar la línea que mejor se adapta a ella. Asimismo, debe usarse en las matrices de series temporales, de modo que se ajuste el resultado del operador make-series. La función genera las columnas siguientes:
* `rsquare`: [r-Square](https://en.wikipedia.org/wiki/Coefficient_of_determination) es una medida estándar de la calidad de ajuste. El valor es un número del intervalo [0-1], donde 1 es el mejor ajuste posible y 0 significa que los datos están desordenados y no se ajustan a ninguna línea. 
* `slope`: Pendiente de la línea aproximada ("a" de y = AX + b).
* `variance`: Varianza de los datos de entrada.
* `rvariance`: Varianza residual que es la varianza entre los valores de los datos de entrada y los aproximados.
* `interception`: Interceptación de la línea aproximada ("b" desde y = AX + b).
* `line_fit`: Matriz numérica que contiene una serie de valores de la mejor línea ajustada. La longitud de la serie es igual que la de la matriz de entrada. Valor de que se usa para los gráficos.

**Sintaxis**

`series_fit_line(`*x1*`)`

**Argumentos**

* *x*: matriz dinámica de valores numéricos.

> [!TIP]
> La manera más cómoda de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print id=' ', x=range(bin(now(), 1h)-11h, bin(now(), 1h), 1h), y=dynamic([2,5,6,8,11,15,17,18,25,26,30,30])
| extend (RSquare,Slope,Variance,RVariance,Interception,LineFit)=series_fit_line(y)
| render timechart
```

:::image type="content" source="images/series-fit-line/series-fit-line.png" alt-text="Línea de ajuste de serie":::

| RSquare | Slope | Variance | RVariance | Interception | LineFit                                                                                     |
|---------|-------|----------|-----------|--------------|---------------------------------------------------------------------------------------------|
| 0.982   | 2.730 | 98.628   | 1.686     | -1.666       | 1,064, 3,7945, 6,526, 9,256, 11,987, 14,718, 17,449, 20,180, 22,910, 25,641, 28,371, 31,102 |

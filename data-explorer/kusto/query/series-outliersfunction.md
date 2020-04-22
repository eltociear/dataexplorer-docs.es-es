---
title: series_outliers() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_outliers() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: 47e479ce7fe09b2456405ac3f7daed4721374f32
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663459"
---
# <a name="series_outliers"></a>series_outliers()

Anota puntos de anomalía en una serie.

Toma una expresión que contiene la matriz numérica dinámica como entrada y genera una matriz numérica dinámica de la misma longitud. Cada valor de la matriz indica una puntuación de posible anomalía utilizando la [prueba de Tukey.](https://en.wikipedia.org/wiki/Outlier#Tukey.27s_test) Un valor mayor que 1,5 o menor que -1,5 indica una anomalía de aumento o disminución respectivamente en el mismo elemento de la entrada.   

**Sintaxis**

`series_outliers(`*x*`, `*tipo*`, `*ignore_val*`, `*min_percentile max_percentile*`, `*max_percentile*`)`

**Argumentos**

* *x*: Celda de matriz dinámica que es una matriz de valores numéricos
* *tipo*: Algoritmo de detección de valores atípicos. Actualmente `"tukey"` es compatible con `"ctukey"` (Tukey tradicional) y (Tukey personalizado). Valor predeterminado: `"ctukey"`
* *ignore_val*: valor numérico que indica que faltan valores en la serie, el valor predeterminado es double(null). La puntuación de valores null `0`e ignore se establece en .
* *min_percentile*: para la calulación del rango de cuantile inter `[2.0, 98.0]` `ctukey` normal, el valor predeterminado es 10, los valores personalizados admitidos están en el rango (solo) 
* *max_percentile:* igual, el valor predeterminado es 90, los valores personalizados admitidos están en el rango `[2.0, 98.0]` (solo ctukey) 

En la tabla siguiente `"tukey"` se `"ctukey"`describen las diferencias entre y:

| Algoritmo | Rango intercuartil predeterminado | Admite rango intercuartil personalizado |
|-----------|----------------------- |--------------------------------|
| `"tukey"` | 25% / 75%              | No                             |
| `"ctukey"`| 10% / 90%              | Sí                            |


> [!TIP]
> La forma más conveniente de utilizar esta función es aplicarla a los resultados del operador [make-series.](make-seriesoperator.md)

**Ejemplo**

Supongamos que tiene una serie temporal con algo de ruido que crea valores atípicos y desea reemplazar esos valores atípicos (ruido) con el valor medio, podría utilizar series_outliers() para detectar los valores atípicos y luego reemplazarlos:

```kusto
range x from 1 to 100 step 1 
| extend y=iff(x==20 or x==80, 10*rand()+10+(50-x)/2, 10*rand()+10) // generate a sample series with outliers at x=20 and x=80
| summarize x=make_list(x),series=make_list(y)
| extend series_stats(series), outliers=series_outliers(series)
| mv-expand x to typeof(long), series to typeof(double), outliers to typeof(double)
| project x, series , outliers_removed=iff(outliers > 1.5 or outliers < -1.5, series_stats_series_avg , series ) // replace outliers with the average
| render linechart
``` 

:::image type="content" border="false" source="images/samples/series-outliers.png" alt-text="Valores atípicos de la serie":::

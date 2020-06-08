---
title: 'series_outliers (): Explorador de datos de Azure'
description: En este artículo se describe series_outliers () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: 80e20e70bc51045f68fd3ef2068f099d750b2b3f
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512442"
---
# <a name="series_outliers"></a>series_outliers()

Puntua los puntos de anomalía en una serie.

La función toma una expresión con una matriz numérica dinámica como entrada y genera una matriz numérica dinámica con la misma longitud. Cada valor de la matriz indica una puntuación de una posible anomalía, mediante ["prueba de Tukey"](https://en.wikipedia.org/wiki/Outlier#Tukey.27s_test). Un valor mayor que 1,5 en el mismo elemento de la entrada indica una anomalía de aumento o disminución. Un valor menor que-1,5 indica una anomalía de rechazo.

**Sintaxis**

`series_outliers(`*x* `, ` *tipo* `, ` de *ignore_val* `, ` *min_percentile* `, ` *max_percentile*`)`

**Argumentos**

* *x*: celda de matriz dinámica que es una matriz de valores numéricos
* *Kind*: algoritmo de detección de valores atípicos. Actualmente admite `"tukey"` (tradicional "Tukey") y `"ctukey"` (personalizado "Tukey"). Valor predeterminado: `"ctukey"`
* *ignore_val*: valor numérico que indica que faltan valores en la serie. El valor predeterminado es Double (NULL). La puntuación de valores NULL y los valores omitidos se establece en`0`
* *min_percentile*: para calcular el intervalo de inter-por cuantiles normal. El valor predeterminado es 10, los valores personalizados admitidos se encuentran en el intervalo `[2.0, 98.0]` ( `ctukey` solo)
* *max_percentile*: igual que el valor predeterminado es 90, los valores personalizados admitidos se encuentran en el intervalo `[2.0, 98.0]` (solo ctukey)

En la tabla siguiente se describen las diferencias entre `"tukey"` y `"ctukey"` :

| Algoritmo | Rango intercuartil predeterminado | Admite rango intercuartil personalizado |
|-----------|----------------------- |--------------------------------|
| `"tukey"` | 25% / 75%              | No                             |
| `"ctukey"`| 10% / 90%              | Yes                            |

> [!TIP]
> La mejor manera de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplo**

Una serie temporal con algún ruido crea valores atípicos. Si desea reemplazar esos valores atípicos (ruido) con el valor medio, use series_outliers () para detectar los valores atípicos y, a continuación, reemplácelos.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 100 step 1 
| extend y=iff(x==20 or x==80, 10*rand()+10+(50-x)/2, 10*rand()+10) // generate a sample series with outliers at x=20 and x=80
| summarize x=make_list(x),series=make_list(y)
| extend series_stats(series), outliers=series_outliers(series)
| mv-expand x to typeof(long), series to typeof(double), outliers to typeof(double)
| project x, series , outliers_removed=iff(outliers > 1.5 or outliers < -1.5, series_stats_series_avg , series ) // replace outliers with the average
| render linechart
``` 

:::image type="content" source="images/series-outliersfunction/series-outliers.png" alt-text="Valores atípicos de la serie" border="false":::

---
title: series_outliers ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe series_outliers () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: 16e82ec68a463b97699f7d02e18c46df65221c7b
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618670"
---
# <a name="series_outliers"></a>series_outliers()

Puntua los puntos de anomalía en una serie.

Toma una expresión que contiene una matriz numérica dinámica como entrada y genera una matriz numérica dinámica con la misma longitud. Cada valor de la matriz indica una puntuación de posible anomalía mediante [la prueba de Tukey](https://en.wikipedia.org/wiki/Outlier#Tukey.27s_test). Un valor mayor que 1,5 o menor que -1,5 indica una anomalía de aumento o disminución respectivamente en el mismo elemento de la entrada.   

**Sintaxis**

`series_outliers(`*x*`, `*Kind*`, `*max_percentile* *min_percentile**ignore_val*ignore_val min_percentile max_percentile`, ``, ``)`

**Argumentos**

* *x*: celda de matriz dinámica que es una matriz de valores numéricos
* *Kind*: algoritmo de detección de valores atípicos. Actualmente admite `"tukey"` (Tukey tradicional) y `"ctukey"` (Custom Tukey). Valor predeterminado: `"ctukey"`
* *ignore_val*: valor numérico que indica que faltan valores en la serie, el valor predeterminado es Double (NULL). La puntuación de valores NULL y los valores omitidos se `0`establece en.
* *min_percentile*: para rango intercuartil del intervalo normal entre por cuantiles, el valor predeterminado es 10, los valores personalizados admitidos `[2.0, 98.0]` se`ctukey` encuentran en el intervalo (solo) 
* *max_percentile*: igual que el valor predeterminado es 90, los valores personalizados admitidos se encuentran en el intervalo `[2.0, 98.0]` (solo ctukey) 

En la tabla siguiente se describen `"tukey"` las `"ctukey"`diferencias entre y:

| Algoritmo | Rango intercuartil predeterminado | Admite rango intercuartil personalizado |
|-----------|----------------------- |--------------------------------|
| `"tukey"` | 25% / 75%              | No                             |
| `"ctukey"`| 10% / 90%              | Sí                            |


> [!TIP]
> La manera más cómoda de usar esta función es aplicarla a los resultados del operador [Make-series](make-seriesoperator.md) .

**Ejemplo**

Supongamos que tiene una serie temporal con algún ruido que crea valores atípicos y desea reemplazar esos valores atípicos (ruido) por el valor medio, podría usar series_outliers () para detectar los valores atípicos y, a continuación, reemplazarlos:

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

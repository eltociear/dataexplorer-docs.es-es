---
title: series_decompose_anomalies ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe series_decompose_anomalies () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/28/2019
ms.openlocfilehash: 51ac499690323b1d2bafb4dc20ab7773f5c99c63
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618933"
---
# <a name="series_decompose_anomalies"></a>series_decompose_anomalies()

Detección de anomalías basada en la descomposición de series (consulte [series_decompose ()](series-decomposefunction.md)) 

Toma una expresión que contiene una serie (matriz numérica dinámica) como entrada y extrae puntos anómalos con puntuaciones.

**Sintaxis**

`series_decompose_anomalies (`*Series* `[, ` *Umbral* *Seasonality* `,` `, ` *Test_points* `,` *Seasonality_threshold* *Trend* de la serie de tendencia estacional Test_points`, ` *AD_method* Seasonality_threshold`,``])`

**Argumentos**

* *Series*: celda de matriz dinámica que es una matriz de valores numéricos, normalmente el resultado de los operadores [Make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md)
* *Umbral*: umbral de anomalías, valor predeterminado 1,5 (valor k) para detectar anomalías leves o más fuertes
* *Estacionalidad*: un entero que controla el análisis estacional, que contiene
    * -1: detección automática de estacionalidad (con [series_periods_detect](series-periods-detectfunction.md)) [predeterminado] 
    * 0: sin estacionalidad (es decir, omitir la extracción de este componente)
    * Period: entero positivo, que especifica el período esperado en número de unidad de compartimientos. Por ejemplo, si la serie está en las bandejas 1H, un período semanal es 168 bandejas
* *Tendencia*: una cadena que controla el análisis de tendencias, que contiene    
    * "AVG": definir el componente de tendencia como promedio de la serie [predeterminado]
    * "ninguna": sin tendencia, omitir la extracción de este componente 
    * "linefit": extraer componente de tendencia mediante regresión lineal
* *Test_points*: 0 [valor predeterminado] o entero positivo, que especifica el número de puntos al final de la serie que se van a excluir del proceso de aprendizaje (regresión). Este parámetro debe establecerse para fines de previsión.
* *AD_method*: una cadena que controla el método de detección de anomalías (vea [series_outliers](series-outliersfunction.md)) en la serie temporal residual, que contiene cualquiera de los dos    
    * "ctukey": [prueba de barrera de Tukey](https://en.wikipedia.org/wiki/Outlier#Tukey's_fences) con intervalo de percentil 10 a 90 personalizado [predeterminado]
    * "Tukey": [prueba de barrera de Tukey](https://en.wikipedia.org/wiki/Outlier#Tukey's_fences) con el intervalo de percentil 25-º estándar
* *Seasonality_threshold*: el umbral de puntuación de estacionalidad cuando la *estacionalidad* está establecida en detección automática, el `0.6` umbral de puntuación predeterminado es (para obtener más detalles, vea [series_periods_detect](series-periods-detectfunction.md)).


**Devolver**

 La función devuelve las siguientes series respectivas:

* `ad_flag`: una serie ternaria que contiene (+ 1,-1,0) marcado/desactivación/sin anomalías, respectivamente
* `ad_score`: puntuación de anomalías
* `baseline`: el valor de predicción de la serie según la descomposición.

**Más información sobre el algoritmo**

Esta función sigue estos pasos:
1. Llama a [series_decompose ()](series-decomposefunction.md) con los parámetros correspondientes para crear la serie de línea de base y de valores residuales.
2. Calcula ad_score serie aplicando [series_outliers ()](series-outliersfunction.md) con el método de detección de anomalías elegido en la serie de valores residuales
3. Calcula la ad_flag serie aplicando el umbral en el ad_score para marcar/inactivo/sin anomalías, respectivamente
 
**Ejemplos**

**1. detección de anomalías en la estacionalidad semanal**

En el ejemplo siguiente, se genera una serie con estacionalidad semanal y luego se agregan algunos valores atípicos. `series_decompose_anomalies`detecta automáticamente la estacionalidad y genera una línea base que captura el patrón repetitivo. Los valores atípicos que agregamos se pueden ver claramente en el ad_score componente.

```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 10.0, 15.0) - (((t%24)/10)*((t%24)/10)) // generate a series with weekly seasonality
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose_anomalies(y)
| render timechart  
```

:::image type="content" source="images/series-decompose-anomaliesfunction/weekly-seasonality-outliers.png" alt-text="Estacionalidad semanal que muestra la línea base y los valores atípicos" border="false":::

**2. detectar anomalías en estacionalidad semanal con tendencia**

En este ejemplo, se agrega una tendencia a la serie del ejemplo anterior. En primer lugar, `series_decompose_anomalies` ejecutamos con los parámetros predeterminados `avg` en los que el valor predeterminado de tendencia solo toma el promedio y no calcula la tendencia, podemos ver que la línea de base generada no contiene la tendencia y es menos precisa en comparación con el ejemplo anterior, por lo que algunos de los valores atípicos que hemos insertado en los datos no se detectan debido a la mayor varianza

```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose_anomalies(y)
| extend series_decompose_anomalies_y_ad_flag = 
series_multiply(10, series_decompose_anomalies_y_ad_flag) // multiply by 10 for visualization purposes
| render timechart   
```

:::image type="content" source="images/series-decompose-anomaliesfunction/weekly-seasonality-outliers-with-trend.png" alt-text="Valores atípicos de estacionalidad semanal con tendencia" border="false":::

A continuación, se ejecuta el mismo ejemplo, pero como se espera una tendencia en la serie, se `linefit` especifica en el parámetro Trend. Podemos ver que la línea de base está mucho más cerca de la serie de entrada. Se detectan todos los valores atípicos que hemos insertado, así como algunos falsos positivos (vea el ejemplo siguiente sobre la optimización del umbral).

```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose_anomalies(y, 1.5, -1, 'linefit')
| extend series_decompose_anomalies_y_ad_flag = 
series_multiply(10, series_decompose_anomalies_y_ad_flag) // multiply by 10 for visualization purposes
| render timechart  
```

:::image type="content" source="images/series-decompose-anomaliesfunction/weekly-seasonality-linefit-trend.png" alt-text="Anomalías semanales de estacionalidad con linefit Trend" border="false":::

**3. ajustar el umbral de detección de anomalías**

En el ejemplo anterior, se detectaron algunos puntos ruidosos como anomalías. en este ejemplo, se aumenta el umbral de detección de anomalías de un valor predeterminado de 1,5 a 2,5 el intervalo de interpercentiles para que solo se detecten anomalías más fuertes. Podemos ver que ahora solo se detectan los valores atípicos que se insertaron en los datos.

```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and onlgoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose_anomalies(y, 2.5, -1, 'linefit')
| extend series_decompose_anomalies_y_ad_flag = 
series_multiply(10, series_decompose_anomalies_y_ad_flag) // multiply by 10 for visualization purposes
| render timechart  
```

:::image type="content" source="images/series-decompose-anomaliesfunction/weekly-seasonality-higher-threshold.png" alt-text="Anomalías de la serie semanal con un umbral de anomalías superior" border="false":::


---
title: series_decompose_anomalies() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_decompose_anomalies() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/28/2019
ms.openlocfilehash: 7d9ca0585b40a97d21690f908a50de5be493c412
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663622"
---
# <a name="series_decompose_anomalies"></a>series_decompose_anomalies()

Detección de anomalías basada en la descomposición en serie (consulte [series_decompose()](series-decomposefunction.md)) 

Toma una expresión que contiene una serie (matriz numérica dinámica) como entrada y extrae puntos anómalos con puntuaciones.

**Sintaxis**

`series_decompose_anomalies (`*Series* `[, ` *Threshold* `,` *Trend* Tendencia`, ` de *la estacionalidad* `,` del umbral de la serie *Test_points* `, ` *AD_method Test_points* `,` *Seasonality_threshold*`])`

**Argumentos**

* *Serie*: Celda de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md)
* *Umbral*: Umbral de anomalía, por defecto 1.5 (valor k) para detectar anomalías leves o más fuertes
* *Estacionalidad*: Un entero que controla el análisis estacional, que contiene
    * -1: autodetect estacionalidad (usando [series_periods_detect](series-periods-detectfunction.md)) [predeterminado] 
    * 0: sin estacionalidad (es decir, omitir la extracción de este componente)
    * período: entero positivo, especificando el período esperado en número de unidades bins. Por ejemplo, si la serie está en contenedores de 1h, un período semanal es de 168 bins
* *Tendencia*: Una cadena que controla el análisis de tendencia, que contiene    
    * "avg": definir el componente de tendencia como promedio de la serie [predeterminado]
    * "none": no hay tendencia, omita la extracción de este componente 
    * "linefit": extrae el componente de tendencia utilizando la regresión lineal
* *Test_points*: 0 [predeterminado] o entero positivo, especificando el número de puntos al final de la serie para excluir del proceso de aprendizaje (regresión). Este parámetro debe establecerse con fines de previsión
* *AD_method*: Una cadena que controla el método de detección de anomalías (consulte [series_outliers)](series-outliersfunction.md)en la serie temporal residual, que contiene    
    * "ctukey": [Prueba de cerca de Tukey](https://en.wikipedia.org/wiki/Outlier#Tukey's_fences) con rango de percentil 10-90 personalizado [predeterminado]
    * "tukey": [Prueba de cerca de Tukey](https://en.wikipedia.org/wiki/Outlier#Tukey's_fences) con rango de percentil estándar 25-75
* *Seasonality_threshold*: El umbral para la puntuación de estacionalidad cuando *Seasonality* está establecido en autodetect, el umbral de puntuación predeterminado es `0.6` (para más detalles, consulte: [series_periods_detect](series-periods-detectfunction.md))


**devolución**

 La función devuelve la siguiente serie respectiva:

* `ad_flag`: una serie ternaria que contiene (+1, -1, 0) marcando arriba/abajo/sin anomalía respectivamente
* `ad_score`: puntuación de anomalía
* `baseline`: el valor previsto de la serie según la descomposición

**Más sobre el algoritmo**

Esta función sigue estos pasos:
1. Llamadas [series_decompose()](series-decomposefunction.md) con los parámetros respectivos para crear la línea base y las series residuales
2. Calcula ad_score serie aplicando [series_outliers()](series-outliersfunction.md) con el método de detección de anomalías elegido en la serie de residuos
3. Calcula la serie ad_flag aplicando el umbral en el ad_score para marcar la anomalía de arriba/abajo/sin anomalías respectivamente
 
**Ejemplos**

**1. Detectar anomalías en la estacionalidad semanal**

En el siguiente ejemplo generamos una serie con estacionalidad semanal, luego le agregamos algunos valores atípicos. `series_decompose_anomalies`detecta automáticamente la estacionalidad y genera una línea base que captura el patrón repetitivo. Los valores atípicos que hemos añadido se pueden detectar claramente en el componente ad_score.

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

:::image type="content" source="images/samples/series-decompose-anomalies1.png" alt-text="Anomalías de descomposición de la serie 1":::

**2. Detectar anomalías en la estacionalidad semanal con tendencia**

En este ejemplo añadimos una tendencia a la serie del ejemplo anterior. En primer `series_decompose_anomalies` lugar, ejecutamos con `avg` los parámetros predeterminados en los que el valor predeterminado de tendencia solo toma el promedio y no calcula la tendencia, podemos ver que la línea base generada no contiene la tendencia y es menos precisa en comparación con el ejemplo anterior, por lo tanto, algunos de los valores atípicos que insertamos en los datos no se detectan debido a la varianza más alta.

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
:::image type="content" source="images/samples/series-decompose-anomalies2.png" alt-text="Anomalías de descomponer la serie 2":::

A continuación, ejecutamos el mismo ejemplo, pero como esperamos una tendencia en la serie, especificamos `linefit` en el parámetro trend. Podemos ver que la línea base está mucho más cerca de la serie de entrada. Se detectan todos los valores atípicos que insertamos, así como algunos falsos positivos (consulte el siguiente ejemplo sobre el ajuste del umbral).

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

:::image type="content" source="images/samples/series-decompose-anomalies3.png" alt-text="Anomalías de descomponer la serie 3":::

**3. Ajustar el umbral de detección de anomalías**

En el ejemplo anterior se detectaron algunos puntos ruidosos como anomalías, en este ejemplo aumentamos el umbral de detección de anomalías de un valor predeterminado de 1,5 a 2,5 el rango de interpercentil para que solo se detecten anomalías más fuertes. Podemos ver que ahora sólo se detectan los valores atípicos que insertamos en los datos.

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

:::image type="content" source="images/samples/series-decompose-anomalies4.png" alt-text="Anomalías de descomponer la serie 4":::

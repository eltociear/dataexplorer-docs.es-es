---
title: 'series_decompose (): Explorador de datos de Azure'
description: En este artículo se describe series_decompose () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: 4500ec5b58c93901e011ea6dd270563d3405ee01
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372862"
---
# <a name="series_decompose"></a>series_decompose()

Aplica una transformación de descomposición en una serie.  

Toma una expresión que contiene una serie (matriz numérica dinámica) como entrada y la descompone en los componentes estacional, Trend y residual.
 
**Sintaxis**

`series_decompose(`*Serie* `[,` de *Estacionalidad* `,` *Tendencia* `,` *Test_points* `,` *Seasonality_threshold*`])`

**Argumentos**

* *Series*: celda de matriz dinámica, que es una matriz de valores numéricos, normalmente la salida resultante de operadores de [creación de serie](make-seriesoperator.md) o de [make_list](makelist-aggfunction.md)
* *Estacionalidad*: un entero que controla el análisis estacional, que contiene
    * -1: detección automática de estacionalidad mediante [series_periods_detect](series-periods-detectfunction.md) (valor predeterminado).
    * Period: entero positivo que especifica el período esperado en número de ubicaciones. Por ejemplo, si la serie está en bandejas de 1 h, un período semanal es 168 bandejas.
    * 0: sin estacionalidad (se omite la extracción de este componente).    
* *Tendencia*: una cadena que controla el análisis de tendencias, que contiene uno de los valores siguientes:
    * "AVG": definir el componente de tendencia como Average (x) (valor predeterminado)
    * "linefit": extraer componente de tendencia mediante la regresión lineal.
    * "none": sin tendencia, se omite la extracción de este componente.    
* *Test_points*: 0 (valor predeterminado) o entero positivo, que especifica el número de puntos al final de la serie que se van a excluir del proceso de aprendizaje (regresión). Este parámetro debe establecerse para fines de previsión.
* *Seasonality_threshold*: el umbral de puntuación de estacionalidad cuando la *estacionalidad* está establecida en detección automática, el umbral de puntuación predeterminado es `0.6` . Para obtener más información, vea [series_periods_detect](series-periods-detectfunction.md).

**Devolver**

 La función devuelve las siguientes series respectivas:

* `baseline`: el valor de predicción de la serie (suma de los componentes estacionales y de tendencia, consulte a continuación).
* `seasonal`: la serie del componente estacional:
    * Si no se detecta el período o se establece explícitamente en 0: constante 0.
    * Si se detecta o se establece en un entero positivo: mediana de los puntos de la serie en la misma fase
* `trend`: la serie del componente de tendencia.
* `residual`: la serie del componente residual (es decir, x-Baseline).
  

**Notas**

* Orden de ejecución de componentes:
    1. Extraer la serie estacional
    2. Reste de x, generando la serie de la temporada
    3. Extraer el componente de tendencia de la serie desestacional
    4. Crear la línea de base = estacional + tendencia
    5. Crear el residual = x-Baseline
    
* Se debe habilitar la estacionalidad y, o la tendencia. De lo contrario, la función es redundante y solo devuelve Baseline = 0 y residual = x.

**Más información sobre la descomposición de series**

Este método se suele aplicar a la serie temporal de métricas que se espera que presenten un comportamiento periódico o de tendencia. Puede usar el método para predecir valores de métricas futuros o detectar valores anómalos. La suposición implícita de este proceso de regresión es que además de la temporada y el comportamiento de tendencia, la serie temporal se estocástico y se distribuye aleatoriamente. Previsión de los valores de métricas futuros de los componentes estacional y Trend, pasando por alto la parte residual. Detecte valores anómalos basados en la detección de valores atípicos solo en la parte residual. Puede encontrar más información en el [capítulo sobre descomposición de series temporales](https://www.otexts.org/fpp/6).

**Ejemplos**

**Estacionalidad semanal**

En el ejemplo siguiente, se genera una serie con estacionalidad semanal y sin tendencia, a continuación, se agregan algunos valores atípicos. `series_decompose`busca y detecta automáticamente la estacionalidad y genera una línea base que es casi idéntica al componente estacional. Los valores atípicos agregados se pueden ver claramente en el componente de residuos.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 10.0, 15.0) - (((t%24)/10)*((t%24)/10)) // generate a series with weekly seasonality
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose(y)
| render timechart  
```

:::image type="content" source="images/samples/series-decompose1.png" alt-text="Descomponer serie 1":::

**Estacionalidad semanal con tendencia**

En este ejemplo, se agrega una tendencia a la serie del ejemplo anterior. En primer lugar, ejecutamos `series_decompose` con los parámetros predeterminados. El `avg` valor predeterminado de tendencia solo toma el promedio y no calcula la tendencia. La línea base generada no contiene la tendencia. Al observar la tendencia en los valores residuales, se hace patente que este ejemplo es menos preciso que el ejemplo anterior.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose(y)
| render timechart  
```

:::image type="content" source="images/samples/series-decompose2.png" alt-text="Descomponer serie 2":::

A continuación, se vuelve a ejecutar el mismo ejemplo. Dado que estamos esperando una tendencia en la serie, especificamos `linefit` en el parámetro Trend. Podemos ver que se ha detectado la tendencia positiva y que la línea base está mucho más cerca de la serie de entrada. Los valores residuales están cercanos a cero y solo se resaltan los valores atípicos. En el gráfico se pueden ver todos los componentes de la serie.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let ts=range t from 1 to 24*7*5 step 1 
| extend Timestamp = datetime(2018-03-01 05:00) + 1h * t 
| extend y = 2*rand() + iff((t/24)%7>=5, 5.0, 15.0) - (((t%24)/10)*((t%24)/10)) + t/72.0 // generate a series with weekly seasonality and ongoing trend
| extend y=iff(t==150 or t==200 or t==780, y-8.0, y) // add some dip outliers
| extend y=iff(t==300 or t==400 or t==600, y+8.0, y) // add some spike outliers
| summarize Timestamp=make_list(Timestamp, 10000),y=make_list(y, 10000);
ts 
| extend series_decompose(y, -1, 'linefit')
| render timechart  
```

:::image type="content" source="images/samples/series-decompose3.png" alt-text="Descomponeción de series 3":::

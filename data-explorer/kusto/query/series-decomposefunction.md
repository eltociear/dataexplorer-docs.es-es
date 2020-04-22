---
title: series_decompose() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_decompose() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: 375d63dd050840cd884fca0198511e71ac46f170
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663610"
---
# <a name="series_decompose"></a>series_decompose()

Aplica una transformación de descomposición en una serie.  

Toma una expresión que contiene una serie (matriz numérica dinámica) como entrada y la descompone en componentes estacionales, de tendencia y residuales.
 
**Sintaxis**

`series_decompose(`*Tendencia* de `[,` *la estacionalidad* `,` *Trend* `,` de la serie *Test_points* `,` *Seasonality_threshold*`])`

**Argumentos**

* *Serie*: Celda de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md)
* *Estacionalidad*: Un entero que controla el análisis estacional, que contiene
    * -1: detectar automáticamente la estacionalidad mediante [series_periods_detect](series-periods-detectfunction.md) (predeterminado).
    * período: entero positivo que especifica el período esperado en número de ubicaciones. Por ejemplo, si la serie está en contenedores de 1h, un período semanal es de 168 ubicaciones.
    * 0: sin estacionalidad (omita la extracción de este componente).    
* *Tendencia*: Una cadena que controla el análisis de tendencia, que contiene uno de los siguientes:
    * "avg": definir el componente de tendencia como average(x) (predeterminado)
    * "linefit": extrae el componente de tendencia utilizando la regresión lineal.
    * "none": no hay tendencia, omita la extracción de este componente.    
* *Test_points*: 0 (predeterminado) o entero positivo, especificando el número de puntos al final de la serie para excluir del proceso de aprendizaje (regresión). Este parámetro debe establecerse para fines de previsión.
* *Seasonality_threshold*: El umbral para la puntuación de estacionalidad cuando *Seasonality* se establece en auto-detect, el umbral de puntuación predeterminado es `0.6`. Para obtener más [información,](series-periods-detectfunction.md)consulte series_periods_detect .

**devolución**

 La función devuelve la siguiente serie respectiva:

* `baseline`: el valor previsto de la serie (suma de componentes estacionales y de tendencia, véase más adelante)
* `seasonal`: la serie del componente estacional:
    * si el período no se detecta o se establece explícitamente en 0: constante 0
    * si se detecta o se establece en entero positivo: mediana de los puntos de serie en la misma fase
* `trend`: la serie del componente de tendencia
* `residual`: la serie del componente residual (es decir, x - línea base)
  

**Notas**

* Orden de ejecución de componentes:
    1. Extraer la serie de temporada
    2. Restarlo de x, generando la serie desestacional
    3. Extraiga el componente de tendencia de la serie desestacional
    4. Crear la línea de base - estacional + tendencia
    5. Crear el residuo x - línea base
    
* Ya sea la estacionalidad y / o tendencia debe estar habilitada, de lo contrario la función es redundante y sólo devuelve la línea de base - 0 y residual - x

**Más información sobre la descomposición de series**

Este método se aplica generalmente a las series temporales de métricas que se espera que manifiesten el comportamiento periódico y/o de tendencia (por ejemplo, tráfico de servicio y otras métricas de uso), con el fin de pronosticar valores de métricas futuros o detectar valores anómalos. La suposición implícita de este proceso de regresión es que aparte del comportamiento estacional y de tendencia (conocido a-priori), la serie temporal es estocástica y distribuida aleatoriamente; consecuentemente, podemos pronosticar valores métricos futuros de los componentes estacionales y de tendencia (ignorando la parte residual), mientras que podemos detectar valores anómalos basados en la detección de valores atípicos solo en la parte residual. Puede encontrar más detalles en el [capítulo Decomposición](https://www.otexts.org/fpp/6) de la Serie Temporal de este gran libro.

**Ejemplos**

**1. Estacionalidad semanal**

En el siguiente ejemplo generamos una serie con estacionalidad semanal y sin tendencia, luego le agregamos algunos valores atípicos. `series_decompose`encuentra auto-detecta la estacionalidad y genera una línea base que es casi idéntica al componente estacional. Los valores atípicos que añadimos se pueden ver claramente en el componente de residuos.

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

:::image type="content" source="images/samples/series-decompose1.png" alt-text="Serie descomponer 1":::

**2. Estacionalidad semanal con tendencia**

En este ejemplo añadimos una tendencia a la serie del ejemplo anterior. En primer `series_decompose` lugar, ejecutamos con `avg` los parámetros predeterminados en los que el valor predeterminado de tendencia sólo toma el promedio y no calcula la tendencia, podemos ver que la línea base generada no contiene la tendencia y es menos precisa en comparación con el ejemplo anterior, es más evidente al observar la tendencia en los residuos.

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

:::image type="content" source="images/samples/series-decompose2.png" alt-text="Serie descomponer 2":::

A continuación, ejecutamos el mismo ejemplo, pero como esperamos una tendencia en la serie, especificamos `linefit` en el parámetro trend. Podemos ver que se detecta la tendencia positiva y la línea de base está mucho más cerca de la serie de entrada. Los residuos están cerca de cero con sólo los valores atípicos destacando. Podemos ver todos los componentes de la serie en el gráfico.

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

:::image type="content" source="images/samples/series-decompose3.png" alt-text="Serie descomponer 3":::

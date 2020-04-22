---
title: Capacidad de aprendizaje automático en Azure Data Explorer
description: Use la agrupación en clústeres de aprendizaje automático para el análisis de causa principal en Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/29/2019
ms.openlocfilehash: e2adedec36696c7974ecb2e7f7c62d65329c368f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494492"
---
# <a name="machine-learning-capability-in-azure-data-explorer"></a>Capacidad de aprendizaje automático en Azure Data Explorer

Azure Data Explorer, una plataforma de análisis de macrodatos, se usa para supervisar el estado del servicio, la QoS o los dispositivos que no funcionan correctamente para detectar comportamientos anómalos mediante funciones integradas de [detección y previsión de anomalías](/azure/data-explorer/anomaly-detection). Una vez detectado un patrón anómalo, se realiza un análisis de causa principal (RCA) para mitigar o resolver la anomalía.

El proceso de diagnóstico, realizado por expertos en la materia, es complejo y largo. El proceso incluye la captura y la combinación de datos adicionales de diferentes orígenes del mismo período de tiempo para buscar cambios en la distribución de valores en varias dimensiones, representar mediante gráficos variables adicionales y aplicar otras técnicas basadas en el conocimiento de la materia y la intuición. Puesto que estos escenarios de diagnóstico son habituales en Azure Data Explorer, hay disponibles complementos de aprendizaje automático para facilitar la fase de diagnóstico y acortar la duración del RCA.

Azure Data Explorer tiene tres complementos de aprendizaje automático: [`autocluster`](kusto/query/autoclusterplugin.md), [`basket`](kusto/query/basketplugin.md) y [`diffpatterns`](kusto/query/diffpatternsplugin.md). Todos los complementos implementan algoritmos de agrupación en clústeres. Los complementos `autocluster` y `basket` agrupan en clústeres un conjunto de registros único y el complemento `diffpatterns` agrupa en clústeres las diferencias entre dos conjuntos de registros.

## <a name="clustering-a-single-record-set"></a>Agrupación en clústeres de un conjunto de registros único

Un escenario habitual es un conjunto de datos seleccionado mediante criterios específicos, como el período de tiempo en que se exhibe el comportamiento anómalo, las lecturas de temperatura alta del dispositivo, los comandos de larga duración y los usuarios que más consumen. Queremos disponer de una manera sencilla y rápida de encontrar patrones comunes (segmentos) en los datos. Los patrones son un subconjunto del conjunto de datos cuyos registros comparten los mismos valores en varias dimensiones (columnas de categorías). A través de la consulta siguiente se genera y se muestra una serie temporal de las excepciones de servicio durante una semana en intervalos de diez minutos:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5XPsaoCQQyF4d6nCFa7oHCtZd9B0F6G8ajByWTJZHS5+PDOgpVgYRn485EkOAnno9NAriWGFKw7QfQYUy0O43zZ0JNKFQnG/5jrbmeIXHBgwd6DjH2/JVqk2QrTL1aYvlifa4tni29YlzaiUK4yRK3Zu54006dBZ1N5/+X6PqpRI23+pFGGfIKRtz5egzk92K+dsycMyz3szhGEKWJ01lxI760O9ABuq0bMcvV2hqFoqnOz7F9BdSHlSgEAAA==) **\]**

```kusto
let min_t = toscalar(demo_clustering1 | summarize min(PreciseTimeStamp));  
let max_t = toscalar(demo_clustering1 | summarize max(PreciseTimeStamp));  
demo_clustering1
| make-series num=count() on PreciseTimeStamp from min_t to max_t step 10m
| render timechart with(title="Service exceptions over a week, 10 minutes resolution")
```

![Gráfico de tiempo de las excepciones de servicio](media/machine-learning-clustering/service-exceptions-timechart.png)

El recuento de excepciones de servicio se correlaciona con el tráfico del servicio total. Puede ver claramente el patrón diario para los días laborables de lunes a viernes, con un aumento del recuento de excepciones de servicio al mediodía y un descenso del recuento durante la noche. A lo largo del fin de semana, pueden verse recuentos bajos planos. Se pueden detectar picos de excepciones mediante la [detección de anomalías de serie temporal](/azure/data-explorer/anomaly-detection?#time-series-anomaly-detection) en Azure Data Explorer.

El segundo pico en los datos se produce el martes por la tarde. La consulta siguiente se utiliza para efectuar un diagnóstico exhaustivo de este pico. Utilice la consulta para volver a dibujar el gráfico en torno al pico en una resolución mayor (ocho horas en intervalos de un minuto) para comprobar si se trata de un pico pronunciado y ver sus bordes.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAAyXNwQrCMBAE0Hu/YvHUooWkghSl/yDoyUsJyWpCk2xJNnjx403pbeYwbzwyBBdnnoxiZBewHYS89GLshzNIeRWiuzUGA83al8yYXPzI5gdBLdjnWjFDLGHSVCK3HVCEe0LtMj4r9mAVVngnCvsLMO3hOFqo2goyVCxhNJhgu9dWJYavY9uyY4/T4UV1XVm2CEM0kFe34AnkBhXGOs7kCzuKh+4P3/XM5M8AAAA=) **\]**

```kusto
let min_t=datetime(2016-08-23 11:00);
demo_clustering1
| make-series num=count() on PreciseTimeStamp from min_t to min_t+8h step 1m
| render timechart with(title="Zoom on the 2nd spike, 1 minute resolution")
```

![Centrarse en el gráfico de tiempo del pico](media/machine-learning-clustering/focus-spike-timechart.png)

Se puede ver un pico de dos minutos estrecho entre las 15:00 y las 15:02. En la consulta siguiente, cuente las excepciones en esta ventana de dos minutos:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA8tJLVHIzcyLL0hNzI4vsU1JLEktycxN1TAyMDTTNbDQNTJWMDS1MjDQtObKASlNrCCk1AioNCU1Nz8+Oae0uCS1KDMv3ZCrRqE8I7UoVSGgKDU5szg1BKgvuCQxt0AhKbWkPDU1TwPhBj09hCWaQI3J+aV5JQACnQoRpwAAAA==) **\]**

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| count
```

|Count |
|---------|
|972    |

En la consulta siguiente, muestree 20 excepciones de un total de 972:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA4XOsQrCMBSF4b1Pccd2aLmJKKL4DoLu4doeNDSJJb1SBx/eOHV0/37OCVCKPrkJMjo9DaJQH1FbNruW963dkNkemJtjFX5U3v+oLXRAfLo+vGZF9uluqg8tD2TQOaP3M66lu6jEiW7QBUj1+qHr1pGmhCojyPIX7QHvzakAAAA=) **\]**

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| take 20
```

| PreciseTimeStamp            | Region | ScaleUnit | DeploymentId                     | Tracepoint | ServiceHost                          |
|-----------------------------|--------|-----------|----------------------------------|------------|--------------------------------------|
| 2016-08-23 15:00:08.7302460 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 100005     | 00000000-0000-0000-0000-000000000000 |
| 2016-08-23 15:00:09.9496584 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 8d257da1-7a1c-44f5-9acd-f9e02ff507fd |
| 2016-08-23 15:00:10.5911748 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 100005     | 00000000-0000-0000-0000-000000000000 |
| 2016-08-23 15:00:12.2957912 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007007   | f855fcef-ebfe-405d-aaf8-9c5e2e43d862 |
| 2016-08-23 15:00:18.5955357 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 9d390e07-417d-42eb-bebd-793965189a28 |
| 2016-08-23 15:00:20.7444854 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 6e54c1c8-42d3-4e4e-8b79-9bb076ca71f1 |
| 2016-08-23 15:00:23.8694999 | eus2   | su2       | 89e2f62a73bb4efd8f545aeae40d7e51 | 36109      | 19422243-19b9-4d85-9ca6-bc961861d287 |
| 2016-08-23 15:00:26.4271786 | ncus   | su1       | e24ef436e02b4823ac5d5b1465a9401e | 36109      | 3271bae4-1c5b-4f73-98ef-cc117e9be914 |
| 2016-08-23 15:00:27.8958124 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 904498     | 8cf38575-fca9-48ca-bd7c-21196f6d6765 |
| 2016-08-23 15:00:32.9884969 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 10007007   | d5c7c825-9d46-4ab7-a0c1-8e2ac1d83ddb |
| 2016-08-23 15:00:34.5061623 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 1002110    | 55a71811-5ec4-497a-a058-140fb0d611ad |
| 2016-08-23 15:00:37.4490273 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 10007006   | f2ee8254-173c-477d-a1de-4902150ea50d |
| 2016-08-23 15:00:41.2431223 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 103200     | 8cf38575-fca9-48ca-bd7c-21196f6d6765 |
| 2016-08-23 15:00:47.2983975 | ncus   | su1       | e24ef436e02b4823ac5d5b1465a9401e | 423690590  | 00000000-0000-0000-0000-000000000000 |
| 2016-08-23 15:00:50.5932834 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007006   | 2a41b552-aa19-4987-8cdd-410a3af016ac |
| 2016-08-23 15:00:50.8259021 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 1002110    | 0d56b8e3-470d-4213-91da-97405f8d005e |
| 2016-08-23 15:00:53.2490731 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 36109      | 55a71811-5ec4-497a-a058-140fb0d611ad |
| 2016-08-23 15:00:57.0000946 | eus2   | su2       | 89e2f62a73bb4efd8f545aeae40d7e51 | 64038      | cb55739e-4afe-46a3-970f-1b49d8ee7564 |
| 2016-08-23 15:00:58.2222707 | scus   | su5       | 9dbd1b161d5b4779a73cf19a7836ebd6 | 10007007   | 8215dcf6-2de0-42bd-9c90-181c70486c9c |
| 2016-08-23 15:00:59.9382620 | scus   | su3       | 90d3d2fc7ecc430c9621ece335651a01 | 10007006   | 451e3c4c-0808-4566-a64d-84d85cf30978 |

### <a name="use-autocluster-for-single-record-set-clustering"></a>Usar un autoclúster para la agrupación en clústeres de un conjunto de registros único

Aunque hay menos de mil excepciones, aún cuesta encontrar segmentos comunes, ya que hay varios valores en cada columna. Puede usar el complemento [`autocluster()`](kusto/query/autoclusterplugin.md) para extraer instantáneamente una pequeña lista de segmentos comunes y buscar los clústeres interesantes en los intervalos de dos minutos del pico, tal como se muestra en la consulta siguiente:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA4WOsQrCMBRF937FG5OhJYkoovQfBN1DbC8aTNqSvlgHP94IQkf3c+65AUzRD3aCe1hue8dgHyGM0rta7WuzIb09KCWPVfii7vUPNQXtEUfbhTwzkh9uunrTckcCnRI6P+NSvDO7ONEVvACDWD80zRqRRcTThVxa5DKPv00hP81KL1+4AAAA) **\]**

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| evaluate autocluster()
```

| SegmentId | Count | Percent | Region | ScaleUnit | DeploymentId | ServiceHost |
|-----------|-------|------------------|--------|-----------|----------------------------------|--------------------------------------|
| 0 | 639 | 65.7407407407407 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 1 | 94 | 9.67078189300411 | scus | su5 | 9dbd1b161d5b4779a73cf19a7836ebd6 |  |
| 2 | 82 | 8.43621399176955 | ncus | su1 | e24ef436e02b4823ac5d5b1465a9401e |  |
| 3 | 68 | 6.99588477366255 | scus | su3 | 90d3d2fc7ecc430c9621ece335651a01 |  |
| 4 | 55 | 5.65843621399177 | weu | su4 | be1d6d7ac9574cbc9a22cb8ee20f16fc |  |

En los resultados anteriores, se puede ver que el segmento predominante contiene el 65,74 % del total de registros de excepciones y comparte cuatro dimensiones. El segmento siguiente es mucho menos frecuente, ya que solo contiene el 9,67 % de los registros y comparte tres dimensiones. Los demás segmentos son incluso menos comunes. 

Un autoclúster utiliza un algoritmo patentado para crear minería de datos de varias dimensiones y extraer segmentos interesantes. "Interesante" significa que cada segmento tiene una cobertura significativa de los conjuntos de registros y de características. Los segmentos también divergen, lo que significa que cada uno de ellos es significativamente diferente de los demás. Uno o más de estos segmentos pueden ser relevantes para el proceso del RCA. Para minimizar la evaluación y revisión de los segmentos, el autoclúster extrae solo una lista de segmentos pequeños.

### <a name="use-basket-for-single-record-set-clustering"></a>Uso de basket() para la agrupación en clústeres de un conjunto de registros único

También puede usar el complemento [`basket()`](kusto/query/basketplugin.md) tal como se muestra en la consulta siguiente:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA4WOsQ6CMBgGd57iH9sB0tZojMZ3MNG9KfBFG1og7Y84+PDWidH9LncBTNGPdoYbLF96x2AfIYzSh1oda7MjvT8pJc9V+KHu/Q81Be0RJ9uFJTOSHx+6+tD6RAJdEzqfcS/ejV2cqQWvwCi2h6bZIrKIeLmwlBa1Lg9gIb9KJv2TswAAAA==) **\]**

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
demo_clustering1
| where PreciseTimeStamp between(min_peak_t..max_peak_t)
| evaluate basket()
```

| SegmentId | Count | Percent | Region | ScaleUnit | DeploymentId | Tracepoint | ServiceHost |
|-----------|-------|------------------|--------|-----------|----------------------------------|------------|--------------------------------------|
| 0 | 639 | 65.7407407407407 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 |  | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 1 | 642 | 66.0493827160494 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 |  |  |
| 2 | 324 | 33.3333333333333 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 | 0 | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 3 | 315 | 32.4074074074074 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 | 16108 | e7f60c5d-4944-42b3-922a-92e98a8e7dec |
| 4 | 328 | 33.7448559670782 |  |  |  | 0 |  |
| 5 | 94 | 9.67078189300411 | scus | su5 | 9dbd1b161d5b4779a73cf19a7836ebd6 |  |  |
| 6 | 82 | 8.43621399176955 | ncus | su1 | e24ef436e02b4823ac5d5b1465a9401e |  |  |
| 7 | 68 | 6.99588477366255 | scus | su3 | 90d3d2fc7ecc430c9621ece335651a01 |  |  |
| 8 | 167 | 17.1810699588477 | scus |  |  |  |  |
| 9 | 55 | 5.65843621399177 | weu | su4 | be1d6d7ac9574cbc9a22cb8ee20f16fc |  |  |
| 10 | 92 | 9.46502057613169 |  |  |  | 10007007 |  |
| 11 | 90 | 9.25925925925926 |  |  |  | 10007006 |  |
| 12 | 57 | 5.8641975308642 |  |  |  |  | 00000000-0000-0000-0000-000000000000 |

Basket implementa el algoritmo Apriori para la minería de conjuntos de elementos y extrae todos los segmentos cuya cobertura del conjunto de registros esté por encima de un umbral (el valor predeterminado es 5 %). Se puede ver que se han extraído más segmentos con coberturas similares (por ejemplo, los segmentos 0,1 o 2,3).

Ambos complementos son eficaces y fáciles de usar, pero su considerable limitación se debe al hecho de que agrupan en clústeres un conjunto de registros único de forma no supervisada (sin etiquetas). Por tanto, no está claro si los patrones extraídos caracterizan el conjunto de registros seleccionado (los registros anómalos) o el conjunto de registros global.

## <a name="clustering-the-difference-between-two-records-sets"></a>Agrupación en clústeres de la diferencia entre dos conjuntos de registros

Con el complemento [`diffpatterns()`](kusto/query/diffpatternsplugin.md) se supera la limitación de `autocluster` y `basket`. `Diffpatterns` toma dos conjuntos de registros y extrae los segmentos principales que son diferentes entre ellos. Por lo general, un conjunto contiene el conjunto de registros anómalo que se está investigando (analizado por `autocluster` y `basket`). El otro conjunto contiene el conjunto de registros de referencia (base de referencia). 

En la consulta siguiente, usamos `diffpatterns` para encontrar clústeres interesantes en los intervalos de dos minutos del pico, que son diferentes de los clústeres de la base de referencia. Definimos el intervalo de la base de referencia como los ocho minutos antes de las 15:00 (cuando se ha iniciado el pico). También debemos ampliar mediante una columna binaria (AB) que especifique si un registro específico pertenece a la base de referencia o al conjunto anómalo. `Diffpatterns` implementa un algoritmo de aprendizaje supervisado, en que la marca anómala, frente a la marca de la base de referencia (AB), ha generado las dos etiquetas de clase.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA42QzU+DQBDF7/wVcwOi5UtrmhJM4OzBRO9kWqbtpssuYacfGv94t0CrxFTd02by5jfvPUkMtVBlQ7gtOauQiUVNXhLFD5NoNknuIJ7Oo8hPHXmS4vEvaXKWWuoCDUmh6Jr8fj79Tv6HfOanEIbwRLgnQFhjAwviA5EC3hCcCYCq6gamEVsC1oB7LfoRt6iMYKEVvGtFQXfeNFKc7mXe2MjNVzl+mARR6lRU63Ipd4apFWodOx9w2FBL4D23tBSGXi3mhbG+OPPGVQTB+ITvg24dGN7vlN5JTxhc+dYAHZls4LzIxGr1k/B4iXcLbq50jfLNtd9i8OB2jD3KnW0dKstokG08Zby8uLbyCfX/tG46AgAA) **\]**

```kusto
let min_peak_t=datetime(2016-08-23 15:00);
let max_peak_t=datetime(2016-08-23 15:02);
let min_baseline_t=datetime(2016-08-23 14:50);
let max_baseline_t=datetime(2016-08-23 14:58); // Leave a gap between the baseline and the spike to avoid the transition zone.
let splitime=(max_baseline_t+min_peak_t)/2.0;
demo_clustering1
| where (PreciseTimeStamp between(min_baseline_t..max_baseline_t)) or
        (PreciseTimeStamp between(min_peak_t..max_peak_t))
| extend AB=iff(PreciseTimeStamp > splitime, 'Anomaly', 'Baseline')
| evaluate diffpatterns(AB, 'Anomaly', 'Baseline')
```

| SegmentId | CountA | CountB | PercentA | PercentB | PercentDiffAB | Region | ScaleUnit | DeploymentId | Tracepoint |
|-----------|--------|--------|----------|----------|---------------|--------|-----------|----------------------------------|------------|
| 0 | 639 | 21 | 65.74 | 1.7 | 64.04 | eau | su7 | b5d1d4df547d4a04ac15885617edba57 |  |
| 1 | 167 | 544 | 17.18 | 44.16 | 26.97 | scus |  |  |  |
| 2 | 92 | 356 | 9.47 | 28,9 | 19.43 |  |  |  | 10007007 |
| 3 | 90 | 336 | 9,26 | 27.27 | 18.01 |  |  |  | 10007006 |
| 4 | 82 | 318 | 8.44 | 25.81 | 17.38 | ncus | su1 | e24ef436e02b4823ac5d5b1465a9401e |  |
| 5 | 55 | 252 | 5.66 | 20.45 | 14.8 | weu | su4 | be1d6d7ac9574cbc9a22cb8ee20f16fc |  |
| 6 | 57 | 204 | 5.86 | 16.56 | 10.69 |  |  |  |  |

El segmento predominante es el mismo segmento que `autocluster` ha extraído; su cobertura en la ventana anómala de dos minutos también es del 65,74 %. Pero su cobertura en la ventana de la base de referencia de ocho minutos solo es del 1,7 %. La diferencia es del 64,04 %. Esta diferencia parece estar relacionada con el pico anómalo. Para comprobar esta suposición, puede dividir el gráfico original en los registros que pertenecen a este segmento problemático frente a los demás segmentos tal como se muestra en la consulta siguiente:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5WRsWrDMBCG9zzF4cmGGuJUjh2Ktw7tUkLTzuEsnRNRnRQkuSQlD185yRTo0EWIO913/J8MRWBttxE6iC5INOhzRey20owhktd2V8EZwsiMXv/Q9Dpfe5I60Idm2kTkQ1E8AczMxMLjf1h4/IN1PzY7Ax0jWQWBdomvhyF/p512FroOMsIxA0zdTdpKn1bHSzmMzbX8TAfjTkw2vqpLp69VpYQaatEogXOBsqrbtl5WDake6yabXWjkv7WkFxeuPGqG5VzWqhQrIUqx6B/L1WKB6aBViy01imT2ANnau94QT9c35xlNVqQAjF9UhpSHAtiRO+lGG/MCUoZ7CTB4x7ePie5mNbk4QDVn6E+ThUT0SQh5iGlM7tHHX4WFgLHOAQAA) **\]**

```kusto
let min_t = toscalar(demo_clustering1 | summarize min(PreciseTimeStamp));  
let max_t = toscalar(demo_clustering1 | summarize max(PreciseTimeStamp));  
demo_clustering1
| extend seg = iff(Region == "eau" and ScaleUnit == "su7" and DeploymentId == "b5d1d4df547d4a04ac15885617edba57"
and ServiceHost == "e7f60c5d-4944-42b3-922a-92e98a8e7dec", "Problem", "Normal")
| make-series num=count() on PreciseTimeStamp from min_t to max_t step 10m by seg
| render timechart
```

![Validación del gráfico de tiempo del segmento "diffpattern"](media/machine-learning-clustering/validating-diffpattern-timechart.png)

Este gráfico nos permite ver que el pico del martes por la tarde se ha debido a las excepciones de este segmento específico, detectadas mediante el complemento `diffpatterns`.

## <a name="summary"></a>Resumen

Los complementos de aprendizaje automático de Azure Data Explorer son útiles para muchos escenarios. `autocluster` y `basket` implementan el algoritmo de aprendizaje no supervisado y son fáciles de usar. `Diffpatterns` implementa el algoritmo de aprendizaje supervisado y, aunque más complejo, es más eficaz a la hora de extraer los segmentos de diferenciación para el RCA.

Estos complementos se utilizan interactivamente en situaciones ad hoc y en servicios de supervisión casi en tiempo real automáticos. En Azure Data Explorer, después de la detección de anomalías en series temporales se produce un proceso de diagnóstico que está muy optimizado para cumplir los estándares de rendimiento necesarios.

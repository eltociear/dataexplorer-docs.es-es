---
title: Previsión y detección de anomalías de serie temporal en Azure Data Explorer
description: Obtenga información sobre cómo analizar datos de series temporales para la previsión y detección de anomalías mediante Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/24/2019
ms.openlocfilehash: e48b3356d01248eb34857c936f9ff2c8dfb7662a
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83374010"
---
# <a name="anomaly-detection-and-forecasting-in-azure-data-explorer"></a>Previsión y detección de anomalías en Azure Data Explorer

Azure Data Explorer realiza la recopilación continua de datos de telemetría desde servicios en la nube o dispositivos de IoT. Estos datos se analizan para obtener diversa información, como la supervisión del estado de los servicios, los procesos físicos de producción, las tendencias de utilización y la previsión de cargas. El análisis se realiza en series temporales de las métricas seleccionadas para buscar un patrón de desviación de la métrica relacionado con su patrón típico de línea de base normal. Azure Data Explorer contiene funcionalidad nativa para crear, manipular y analizar múltiples series temporales. Puede crear y analizar miles de series temporales en cuestión de segundos, lo que permite obtener flujos de trabajo y soluciones de supervisión casi en tiempo real.

En este artículo se detallan las capacidades de Azure Data Explorer de previsión y detección de anomalías de series temporales. Las funciones de serie temporal aplicables se basan en un modelo de descomposición conocido sólido, en el que cada serie temporal original se descompone en los componentes estacional, residual y de tendencia. Las anomalías se detectan mediante los valores atípicos en el componente residual, mientras que la previsión se realiza mediante la extrapolación de los componentes estacional y de tendencia. La implementación de Azure Data Explorer mejora significativamente el modelo de descomposición básico por detección de estacionalidad automática, análisis de valores atípicos sólido e implementación vectorizada para procesar miles de series temporales en cuestión de segundos.

## <a name="prerequisites"></a>Prerrequisitos

Lea [Análisis de series temporales en Azure Data Explorer](time-series-analysis.md) para obtener una introducción a las capacidades de serie temporal.

## <a name="time-series-decomposition-model"></a>Modelo de descomposición de series temporales

La implementación nativa de Azure Data Explorer para la predicción y detección de anomalías de serie temporal utiliza un modelo de descomposición conocido. Este modelo se aplica a series temporales de métricas que se espera que manifiesten un comportamiento periódico y de tendencia, como el tráfico del servicio, latidos del componente y medidas periódicas de IoT para predecir valores de métrica futuros y detectar los valores anómalos. La suposición de este proceso de regresión es que, con la excepción del comportamiento estacional y de tendencia anteriormente conocido, la serie temporal se distribuye de forma aleatoria. Por consiguiente, puede predecir los valores de métrica futuros a partir de los componentes estacional y de tendencia, denominados colectivamente como línea de base, y omitir la parte residual. También puede detectar valores anómalos según el análisis de valores atípicos utilizando solo la parte residual.
Para crear un modelo de descomposición, use la función [`series_decompose()`](kusto/query/series-decomposefunction.md). La función `series_decompose()` toma un conjunto de series temporales y descompone automáticamente cada serie temporal en sus componentes estacional, de tendencia, residual y de línea de base. 

Por ejemplo, puede descomponer el tráfico de un servicio web interno utilizando la consulta siguiente:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA3WQ3WrDMAyF7/sUukvCnDXJGIOVPEULuwxqoixm/gm2+jf28JObFjbYrmyho3M+yRCD1a5jaGFAJtaW8qaqX8qqLqvnYrMySYHnvxRNWT1B07xW1U03JFEzbVYDWd9Z/KAuUtAUm9UXpLJcSnAH2+LxPZe3AO9gJ6ZbRjvDGLy9EbG/BUemOXnvLxD1AOJ1mijQtWhbyHbbOgOA9RogkqGeAaXn3g1BooVb6OiDNHpD6CjAUccDGv2JrL0TSzozuQHyPYqHdqRkDKN3aBRwkJaCQJIoQ4VsuXh2A/Xezj5SWkVBWSvI0vSoOSsWpLtEpyDwY4KTW8nnJ5ws+2+eAhSyOxjkd+HDVVcIfHplp2TYTxgYTpqnnDUbarM32gPO86PY4jjqfmGw3vGkftNlCi5xNprbWW5kYvENQQnqDh8CAAA=) **\]**

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend (baseline, seasonal, trend, residual) = series_decompose(num, -1, 'linefit')  //  decomposition of a set of time series to seasonal, trend, residual, and baseline (seasonal+trend)
| render timechart with(title='Web app. traffic of a month, decomposition', ysplit=panels)
```

![Descomposición de series temporales](media/anomaly-detection/series-decompose-timechart.png)

* La serie temporal original se denomina **num** (en rojo). 
* El proceso empieza por la detección automática de la estacionalidad mediante la función [`series_periods_detect()`](kusto/query/series-periods-detectfunction.md) y extrae el patrón **estacional** (en púrpura).
* El patrón estacional se resta de la serie temporal original y se ejecuta una regresión lineal con la función [`series_fit_line()`](kusto/query/series-fit-linefunction.md) para encontrar el componente de **tendencia** (en azul claro).
* La función resta la tendencia y lo que queda es el componente **residual** (en verde).
* Por último, la función agrega los componentes estacional y de tendencia para generar la **línea de base** (en azul).

## <a name="time-series-anomaly-detection"></a>Detección de anomalías en una serie temporal

La función [`series_decompose_anomalies()`](kusto/query/series-decompose-anomaliesfunction.md) busca puntos anómalos en un conjunto de series temporales. Esta función llama a `series_decompose()` para generar el modelo de descomposición y, a continuación, ejecuta [`series_outliers()`](kusto/query/series-outliersfunction.md) en el componente residual. `series_outliers()` calcula las puntuaciones de anomalía para cada punto del componente residual mediante la prueba de la barrera de Tukey. Las puntuaciones de anomalía por encima de 1,5 o por debajo de -1,5 indican un aumento o una disminución leves de las anomalías, respectivamente. Las puntuaciones de anomalía por encima de 3,0 o por debajo de -3,0 indican una anomalía segura. 

La siguiente consulta le permite detectar anomalías en el tráfico de un servicio web interno:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA3WR3W7CMAyF73mKI25KpRbaTmjSUJ8CpF1WoXVptPxUifmb9vBLoGO7GFeR7ePv2I4ihpamYdToBBNLTYuqKF/zosyLdbqZqagQl/8UVV68oKreimLSdVFUDZtZR9o2WnxQ48lJ8tXsCzHM7yHMUdfidFiEN4U12AXoloUe0Turp4nYTsaeaYzs/RVedgis80CObkFdI9ltywTAagV4UtQyRKiZgyLEaTGZ9taFQqtIGHI4SX8USn4KltYEJF2YTIeFMFaHPPkMvrWOMuxFoEpDaVjujmo6aq0erafmIY+7ZCiX6wx5mSGJHb3kJA1sF8jB8q69toNwjLPkYfGTseqoja//eLNkRXXyTnuIcVyCneh72cL2YQdtDQ8ZHvIkDcsfPWH+3AvPvObx0FMXD/RLhfDYW9VhtNKwj/8U69M1b2S//AbRUQMWQQIAAA==) **\]**

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend (anomalies, score, baseline) = series_decompose_anomalies(num, 1.5, -1, 'linefit')
| render anomalychart with(anomalycolumns=anomalies, title='Web app. traffic of a month, anomalies') //use "| render anomalychart with anomalycolumns=anomalies" to render the anomalies as bold points on the series charts.
```

![Detección de anomalías en una serie temporal](media/anomaly-detection/series-anomaly-detection.png)

* La serie temporal original (en rojo). 
* El componente de línea de base (estacional + tendencia) (en azul).
* Los puntos anómalos (en púrpura) encima de la serie temporal original. Los puntos anómalos se desvían significativamente de los valores esperados de la línea de base.

## <a name="time-series-forecasting"></a>Previsión de series temporales

La función [`series_decompose_forecast()`](kusto/query/series-decompose-forecastfunction.md) predice valores futuros de un conjunto de series temporales. Esta función llama a `series_decompose()` para generar el modelo de descomposición y, a continuación, para cada serie temporal, extrapola el componente de línea de base al futuro.

La consulta siguiente le permite predecir el tráfico del servicio web de la próxima semana:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA22QzW6DMBCE73mKuQFqKISqitSIW98gkXpEDl5iK9hG9uanUR++dqE99YRGO8x845EYRtuO0UIKJtaG8qbebMt6U9avxW41Joe4/+doyvoFTfNW14tPJlOjZqGc1w9n263crSQZ1xlxpi6Q1xSa1ReSLGcJezGtuJ7y+C3gLA6xZM/CTBi8MwshuxnkaUlGYJpS5/ETQUvEzJsiTz+ibZEd9psMQFUBgUbqGSLe7GkkpBVYygfn46EfSVjyuOpwEaN+CNbOxki6M1mZTNSLkAbOv3WSemcmF6j7vSX8dcTUlvOFsZJcFDHFx4wYnmp7JTzjplnlrHmkNvugI8Q0PYO9GAbdww0RyDjLav1XHLnBimAjEG5E5zQ7vRP284x36hOOTtxZ8Q3The8P2QEAAA==) **\]**

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
let horizon=7d;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t+horizon step dt by sid 
| where sid == 'TS1'   //  select a single time series for a cleaner visualization
| extend forecast = series_decompose_forecast(num, toint(horizon/dt))
| render timechart with(title='Web app. traffic of a month, forecasting the next week by Time Series Decomposition')
```

![Previsión de series temporales](media/anomaly-detection/series-forecasting.png)

* Métrica original (en rojo). Faltan los valores futuros, que se establecen en 0 de manera predeterminada.
* Extrapole el componente de línea de base (en azul) para predecir los valores de la próxima semana.

## <a name="scalability"></a>Escalabilidad

La sintaxis del lenguaje de consulta de Azure Data Explorer permite que una única llamada procese varias series temporales. Su implementación optimizada única permite un rendimiento rápido, lo que es vital para una previsión y detección de anomalías eficaz si se supervisan miles de contadores en escenarios casi en tiempo real.

La consulta siguiente muestra el procesamiento simultáneo de tres series temporales:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA21Qy26DMBC85yvmFlChcUirSI34ikTqETl4KVawjfDmqX587UCaHuqLtePxPLYjhtG2YpRQkom1oaQQy3Uulrl4TzezLjLk5T9GkYsViuJDiImnIqlox6F1g745W67VZqbIuMrIA1WeBk2+mH0jjvk4wh5NKU9fSbhTOItdMNmyND2awZkpIbsxyMukDM/UR8/9FV6rIEkXJqvgmsYTl7X0lISHspzvtqt5hjdxPxkeYBHA4gGKFMBiAUilIAfWja617CY1NG4ASX/FSfuj7PRNsg4ZXANz7Fj3HSGuBmOjZ5hYbcSqIBwbZpNk+iQFcQpx4/omrqLamd55qh5v41d22nIybWChOI0qQ9Cg4e5ftyE6zprbhDV3VM4/aQ/Z96/gQTahU4wsYZzlNvs11vYL3BJsCIQz0eHed/W30jz9AUEBI0ktAgAA) **\]**

```kusto
let min_t = datetime(2017-01-05);
let max_t = datetime(2017-02-03 22:00);
let dt = 2h;
let horizon=7d;
demo_make_series2
| make-series num=avg(num) on TimeStamp from min_t to max_t+horizon step dt by sid
| extend offset=case(sid=='TS3', 4000000, sid=='TS2', 2000000, 0)   //  add artificial offset for easy visualization of multiple time series
| extend num=series_add(num, offset)
| extend forecast = series_decompose_forecast(num, toint(horizon/dt))
| render timechart with(title='Web app. traffic of a month, forecasting the next week for 3 time series')
```

![Escalabilidad de las series temporales](media/anomaly-detection/series-scalability.png)

## <a name="summary"></a>Resumen

En este documento se detallan las funciones nativas de Azure Data Explorer para la previsión y detección de anomalías en series temporales. Cada serie temporal original se descompone en los componentes estacional, de tendencia y residual para detectar anomalías y/o realizar previsiones. Estas funcionalidades se pueden usar para escenarios de supervisión casi en tiempo real, como la detección de errores, el mantenimiento predictivo y la previsión de cargas y peticiones.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre las [capacidades de aprendizaje automático](machine-learning-clustering.md) en Azure Data Explorer.

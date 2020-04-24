---
title: Análisis de datos de series temporales mediante Azure Data Explorer
description: Aprenda a analizar datos de series temporales en la nube mediante Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/07/2019
ms.openlocfilehash: 600c5421af21f8d10f3575854a38f1d95ec27160
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492404"
---
# <a name="time-series-analysis-in-azure-data-explorer"></a>Análisis de series temporales en Azure Data Explorer

Azure Data Explorer realiza la recopilación continua de datos de telemetría desde dispositivos en la nube o dispositivos de IoT. Estos datos se pueden analizar para obtener diversa información, como la supervisión del estado de los servicios, los procesos físicos de producción y las tendencias de utilización. El análisis se realiza en series temporales de métricas seleccionadas para encontrar una desviación en el patrón comparado con su patrón típico de línea de base.
Azure Data Explorer contiene compatibilidad nativa para la creación, manipulación y análisis de varias series temporales. En este tema, aprenderá cómo se usa Azure Data Explorer para crear y analizar **miles de serie temporales en segundos**, lo que permite flujos de trabajo y soluciones de supervisión casi en tiempo real.

## <a name="time-series-creation"></a>Creación de series temporales

En esta sección, vamos a crear un gran conjunto de series temporales regulares de forma sencilla e intuitiva mediante el operador `make-series` y vamos a rellenar los valores que faltan según sea necesario.
El primer paso en el análisis de series temporales es dividir y transformar la tabla de telemetría original en un conjunto de series temporales. La tabla generalmente contiene una columna de marca de tiempo, dimensiones contextuales y métricas opcionales. Las dimensiones se usan para particionar los datos. El objetivo es crear miles de series temporales por partición a intervalos de tiempo regulares.

La tabla de entrada *demo_make_series1* contiene registros de 600 K de tráfico del servicio web arbitrario. Utilice el siguiente comando para muestrear 10 registros:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA0tJzc2Pz03MTo0vTi3KTC02VKhRKAFyFQwNADOyzKUbAAAA) **\]**

```kusto
demo_make_series1 | take 10 
```

La tabla resultante contiene una columna de marca de tiempo, tres columnas de dimensiones contextuales y ninguna métrica:

|   |   |   |   |   |
| --- | --- | --- | --- | --- |
|   | TimeStamp | BrowserVer | OsVer | País/región |
|   | 2016-08-25 09:12:35.4020000 | Chrome 51.0 | Windows 7 | Reino Unido |
|   | 2016-08-25 09:12:41.1120000 | Chrome 52.0 | Windows 10 |   |
|   | 2016-08-25 09:12:46.2300000 | Chrome 52.0 | Windows 7 | Reino Unido |
|   | 2016-08-25 09:12:46.5100000 | Chrome 52.0 | Windows 10 | Reino Unido |
|   | 2016-08-25 09:12:46.5570000 | Chrome 52.0 | Windows 10 | República de Lituania |
|   | 2016-08-25 09:12:47.0470000 | Chrome 52.0 | Windows 8.1 | India |
|   | 2016-08-25 09:12:51.3600000 | Chrome 52.0 | Windows 10 | Reino Unido |
|   | 2016-08-25 09:12:51.6930000 | Chrome 52.0 | Windows 7 | Países Bajos |
|   | 2016-08-25 09:12:56.4240000 | Chrome 52.0 | Windows 10 | Reino Unido |
|   | 2016-08-25 09:13:08.7230000 | Chrome 52.0 | Windows 10 | India |

Dado que no hay métricas, solo podemos crear un conjunto de series temporales que representen el mismo recuento de tráfico, particionado por el sistema operativo mediante la siguiente consulta:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5XPwQrCMBAE0Hu/Yo4NVLBn6Td4ULyWtV1tMJtIsoEq/XhbC4J48jgw+5h1rBDrW0UDDakjR7HsWUIrdOM2cbScakxIWYSiffJSL49W+KAkd2N2hVsMGv8yaPw2furFhCVu1gifpelC9loa9Hyh7LTZInh8FFiPSP7K5fufap1UoR4Mzg/s04njjEb2PUfofNYNFPUFtJiguAEBAAA=) **\]**

```kusto
let min_t = toscalar(demo_make_series1 | summarize min(TimeStamp));
let max_t = toscalar(demo_make_series1 | summarize max(TimeStamp));
demo_make_series1
| make-series num=count() default=0 on TimeStamp in range(min_t, max_t, 1h) by OsVer
| render timechart 
```

- Utilice el operador [`make-series`](kusto/query/make-seriesoperator.md) para crear un conjunto de tres series temporales, donde:
    - `num=count()`: serie temporal de tráfico
    - `range(min_t, max_t, 1h)`: la serie temporal se crea en intervalos de 1 hora en el intervalo de tiempo (marcas de tiempo más recientes y más antiguas de los registros de la tabla)
    - `default=0`: especifique el método de relleno para los intervalos que faltan para crear series temporales regulares. O bien use [`series_fill_const()`](kusto/query/series-fill-constfunction.md), [`series_fill_forward()`](kusto/query/series-fill-forwardfunction.md), [`series_fill_backward()`](kusto/query/series-fill-backwardfunction.md) y [`series_fill_linear()`](kusto/query/series-fill-linearfunction.md) para los cambios
    - `byOsVer`: partición por sistema operativo
- La estructura de datos de series temporales reales es una matriz numérica del valor agregado por cada intervalo temporal. Usamos `render timechart` para la visualización.

En la tabla anterior, tenemos tres particiones. Podemos crear una serie temporal independiente: Windows 10 (rojo), 7 (azul) y 8.1 (verde) para cada versión de sistema operativo, tal y como se muestra en el gráfico:

![Partición de series temporales](media/time-series-analysis/time-series-partition.png)

## <a name="time-series-analysis-functions"></a>Funciones de análisis de series temporales

En esta sección, realizaremos las funciones típicas de procesamiento en serie.
Cuando se ha creado un conjunto de series temporales, Azure Data Explorer admite una lista creciente de funciones para procesar y analizarlas que se encuentran en la [documentación de series temporales](kusto/query/machine-learning-and-tsa.md). Describiremos algunas funciones representativas para el procesamiento y análisis de series temporales.

### <a name="filtering"></a>Filtros

El filtrado es una práctica común en el procesamiento de señales y muy útil para tareas de procesamiento de series temporales (por ejemplo, suavizar una señal con ruido o detectar cambios).
- Hay dos funciones de filtrado genéricas:
    - [`series_fir()`](kusto/query/series-firfunction.md): Aplicación de filtro FIR. Se utiliza para el cálculo simple de la media acumulada y la diferenciación de las series temporales para la detección de cambios.
    - [`series_iir()`](kusto/query/series-iirfunction.md): Aplicación de filtro IIR. Se utiliza para el suavizado exponencial y la suma acumulativa.
- `Extend` la serie temporal establecida mediante la adición de una nueva serie de media acumulada de intervalos de tamaño 5 (denominada *ma_num*) a la consulta:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5WPQavCMBCE7/6KOSYQ4fXgSfobPDx517C2q4bXpLLZQBV/vKkFQTx5WRh25tvZgRUxJK9ooWPuaCAxPcfRR/pnn1kC5wZ35BIjSbjxbDf7EPlXKV6s3a6GmUHTVwya3hkf9tUds1wvEqnEthtLUmPR85HKoO0PxoQXBSFBKJ3YPP9xSyWH5mxxuGKX/1gqlCfl1Neln5EL3R+DmCodhC9MahqHjXVQKbxMW5NScyzQerA7k+gDa1tswzsBAAA=) **\]**

```kusto
let min_t = toscalar(demo_make_series1 | summarize min(TimeStamp));
let max_t = toscalar(demo_make_series1 | summarize max(TimeStamp));
demo_make_series1
| make-series num=count() default=0 on TimeStamp in range(min_t, max_t, 1h) by OsVer
| extend ma_num=series_fir(num, repeat(1, 5), true, true)
| render timechart
```

![Filtrado de series temporales](media/time-series-analysis/time-series-filtering.png)

### <a name="regression-analysis"></a>Análisis de regresión

Azure Data Explorer admite el análisis de regresión lineal segmentado para calcular la tendencia de la serie temporal.
- Utilice [series_fit_line()](kusto/query/series-fit-linefunction.md) para ajustar la mejor línea a una serie temporal para la detección general de tendencias.
- Utilice [series_fit_2lines()](kusto/query/series-fit-2linesfunction.md) para detectar cambios de tendencia, en relación con la línea de base, que sean útiles en los escenarios de supervisión.

Ejemplo de las funciones `series_fit_line()` y `series_fit_2lines()` en una consulta de serie temporal:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA0tJzc2PL04tykwtNuKqUUitKEnNS1GACMSnZZbEG+Vk5qUWa1Rq6iCLggSBYkAdRUD1qUUKIIHkjMSiEoXyzJIMjYrk/JzS3DzbCk0AUIIJ02EAAAA=) **\]**

```kusto
demo_series2
| extend series_fit_2lines(y), series_fit_line(y)
| render linechart with(xcolumn=x)
```

![Regresión de serie temporal](media/time-series-analysis/time-series-regression.png)

- Azul: serie temporal original
- Verde: línea ajustada
- Rojo: dos líneas ajustadas

> [!NOTE]
> La función ha detectado con precisión el punto de salto (cambio de nivel).

### <a name="seasonality-detection"></a>Detección de estacionalidad

Muchas métricas siguen los patrones estacionales (periódicos). El tráfico de usuarios de los servicios en la nube suele contener patrones diarios y semanales que son más altos en la mitad del día laborable y más bajos por la noche y durante el fin de semana. Medida de sensores de IoT en intervalos periódicos. Las medidas físicas, como temperatura, presión y humedad también pueden mostrar el comportamiento estacional.

En el ejemplo siguiente se aplica la detección de estacionalidad en el tráfico de un mes de un servicio web (intervalos de 2 horas):

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA0tJzc2PL04tykwtNuaqUShKzUtJLVIoycxNTc5ILCoBAHrjE80fAAAA) **\]**

```kusto
demo_series3
| render timechart 
```

![Estacionalidad de series temporales](media/time-series-analysis/time-series-seasonality.png)

- Utilice [series_periods_detect()](kusto/query/series-periods-detectfunction.md) para detectar automáticamente los períodos de la serie temporal. 
- Utilice [series_periods_validate()](kusto/query/series-periods-validatefunction.md) si sabemos que una métrica debe tener uno o varios períodos distintos específicos y queremos comprobar su existencia.

> [!NOTE]
> Es una anomalía si no existen períodos específicos distintos.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA12OwQ6CMBBE737FHKmpVtAr39IguwkYyzZ0IZj48TZSLx533szOEAfxieeR0/XwRpzlwb2iilkSShapl5mTQYvd5QvxxJqd1bQEi8vZor6RawaLxsA5FewcOjBKBOP0PXUMXL7lyrCeeIvdRPjrzIw35Qyoe6W2GY4qJMv9yb91xtX0AS7N323BAAAA) **\]**

```kusto
demo_series3
| project (periods, scores) = series_periods_detect(num, 0., 14d/2h, 2) //to detect the periods in the time series
| mv-expand periods, scores
| extend days=2h*todouble(periods)/1d
```

|   |   |   |   |
| --- | --- | --- | --- |
|   | periods | scores | days |
|   | 84 | 0.820622786055595 | 7 |
|   | 12 | 0.764601405803502 | 1 |

La función detecta la estacionalidad diaria y semanal. La puntuación diaria es menor que la semanal porque los días de fin de semana son diferentes de los días de la semana.

### <a name="element-wise-functions"></a>Funciones por elemento

Las operaciones aritméticas y lógicas pueden realizarse en una serie temporal. Con [series_subtract()](kusto/query/series-subtractfunction.md) podemos calcular una serie temporal residual, es decir, la diferencia entre la métrica sin procesar original y una suavizada, y buscar anomalías en la señal residual:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5WQQU/DMAyF7/sVT5waqWjrgRPqb+AAgmPltR6LSNLJcdhA+/G4izRAnLhEerbfl2cHVkSfBkUPnfNIgaSZOM5DpDceMovn3OGMXGIk8Z+8jDdPPvKjUjw4d78KC4NO/2LQ6Tfjz/jqjEXeVolUYj/OJWnjMPGOStB+gznhSoFPEEqv3Fz2aWukFt3eYfuBh/zMYlA+KafJmsOCrPRh56Ux2UL4wKRN1+LOtVApXF/37RTOfioUfvpz2arQqBVS2Q7rtc6wa4wlkPLVCLXIqE7DHvcsXOOh73Hz4tM0HzO6zQ1gDOx8UOvZrtayst0Y7z4babkkYQxMyQbGPYnCiGIxTS/fXGpfwk+n7uQBAAA=) **\]**

```kusto
let min_t = toscalar(demo_make_series1 | summarize min(TimeStamp));
let max_t = toscalar(demo_make_series1 | summarize max(TimeStamp));
demo_make_series1
| make-series num=count() default=0 on TimeStamp in range(min_t, max_t, 1h) by OsVer
| extend ma_num=series_fir(num, repeat(1, 5), true, true)
| extend residual_num=series_subtract(num, ma_num) //to calculate residual time series
| where OsVer == "Windows 10"   // filter on Win 10 to visualize a cleaner chart 
| render timechart
```

![Operaciones en series temporales](media/time-series-analysis/time-series-operations.png)

- Azul: serie temporal original
- Rojo: series temporales suavizadas
- Verde: series temporales residuales

## <a name="time-series-workflow-at-scale"></a>Flujo de trabajo de series temporales a escala

En el siguiente ejemplo se muestra cómo estas funciones pueden ejecutarse a escala en miles de series temporales en segundos para la detección de anomalías. Para ver algunos registros de telemetría de ejemplo de una métrica de recuento de lecturas de un servicio de base de datos durante cuatro días, ejecute la siguiente consulta:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA0tJzc2Pz03Mq4wvTi3KTC025KpRKEnMTlUwAQArfAiiGgAAAA==) **\]**

```kusto
demo_many_series1
| take 4 
```

|   |   |   |   |   |   |
| --- | --- | --- | --- | --- | --- |
|   | timestamp | Loc | anonOp | DB | DataRead |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | 5117853934049630089 | 262 | 0 |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | 5117853934049630089 | 241 | 0 |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | -865998331941149874 | 262 | 279862 |
|   | 2016-09-11 21:00:00.0000000 | Loc 9 | 371921734563783410 | 255 | 0 |

Y estadísticas sencillas:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA0tJzc2Pz03Mq4wvTi3KTC025KpRKC7NzU0syqxKVcgrzbVNzi/NK9HQ1FHIzcyLL7EFkhohnr6uwSGOvgEg0cQKkGhiBZIoAEq2dK9VAAAA) **\]**

```kusto
demo_many_series1
| summarize num=count(), min_t=min(TIMESTAMP), max_t=max(TIMESTAMP) 
```

|   |   |   |   |
| --- | --- | --- | --- |
|   | num | min\_t | max\_t |
|   | 2177472 | 2016-09-08 00:00:00.0000000 | 2016-09-11 23:00:00.0000000 |

La creación de una serie temporal en intervalos de 1 hora de la métrica de lectura (total de cuatro días * 24 horas = 96 puntos) da como resultado una fluctuación normal del patrón:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5WPMQvCMBSE9/6KGxOoYGfpIOjgUBDtXh7twwabFF6ittIfb2rBQSfHg+8+7joOsMZVATlC72vqSFTDtq8subHyLIZ9hgn+Zi2JefKMq/JQ7M/ltjhqvQGSbrbQ8JeFhm/LTyGZInbl1RIhTI3P6X5ROwp0ikmjd/hYYByE3IXV+1G6TEqRtTqahF3DgmAs1y1JwMOEVo0Rzdf6BbBH5FAHAQAA) **\]**

```kusto
let min_t = toscalar(demo_many_series1 | summarize min(TIMESTAMP));  
let max_t = toscalar(demo_many_series1 | summarize max(TIMESTAMP));  
demo_many_series1
| make-series reads=avg(DataRead) on TIMESTAMP in range(min_t, max_t, 1h)
| render timechart with(ymin=0) 
```

![Serie temporal a escala](media/time-series-analysis/time-series-at-scale.png)

El comportamiento anterior es engañoso, ya que la única serie temporal normal se agrega a partir de miles de instancias diferentes que pueden tener patrones anormales. Por lo tanto, creamos una serie temporal por instancia. Una instancia se define por Loc (ubicación), anonOp (operación) y DB (máquina específica).

¿Cuántas series temporales podemos crear?

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA0tJzc2Pz03Mq4wvTi3KTC025KpRKC7NzU0syqxKVUiqVPDJT9ZR8C/QUXBxAkol55fmlQAAWEsFxjQAAAA=) **\]**

```kusto
demo_many_series1
| summarize by Loc, Op, DB
| count
```

|   |   |
| --- | --- |
|   | Count |
|   | 18339 |

Ahora, vamos a crear un conjunto de 18339 series temporales de la métrica de recuento de lecturas. Agregamos la cláusula `by` a la instrucción make-series, aplicamos la regresión lineal y seleccionamos las dos primeras series temporales que tuvieron la tendencia decreciente más significativa:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5WPsU7DQBBE+3zFdLmTTGHSgFAKUCiQiIKIe2u5rJ0T9l3YWwcH5eO5JBIFVJSzmnmz07Gi96FWzKExOepIzIb7WPcUDnVi8ZxKHJGGvifxX3yym+pp+biu7pcv1t4Bk+5EofFfFBp/U/4EJsdse+eri4QwbdKc9q1ZkNJrVhYx4IcCHyAUWjbnRcXlpQLl1uLtgOfoCqx2BRYPGcyjctjASPoYSLhA6uKObR5waasbr3XnA5tzrc0RjTtcn0hnKyg55KtkDAvU9+y2JIpPr1ujXjueT9cse+8YlVDTeIfVoNQymiiZ5ENSCi4vM3FQxAblzWx2a6f2G2UcBRyWAQAA) **\]**

```kusto
let min_t = toscalar(demo_many_series1 | summarize min(TIMESTAMP));  
let max_t = toscalar(demo_many_series1 | summarize max(TIMESTAMP));  
demo_many_series1
| make-series reads=avg(DataRead) on TIMESTAMP in range(min_t, max_t, 1h) by Loc, Op, DB
| extend (rsquare, slope) = series_fit_line(reads)
| top 2 by slope asc 
| render timechart with(title='Service Traffic Outage for 2 instances (out of 18339)')
```

![Dos primeras series temporales](media/time-series-analysis/time-series-top-2.png)

Mostrar las instancias:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA5WPvW4CMRCEe55iSlsyBWkjChApIoESAb21udsQg38O26AD8fDx3SEUJVXKWc18s2M5wxmvM6bIIVVkKYqaXdCO/EUnjobTBDekk3MUzZU7u9i+rl4229nqXcpnYGQ7CrX/olD7m/InMLoV24HHg0RkqtOUzjuxoEzroiSCx4MC4xHJ71j0i9TwksLkS+LjgmWoFN4ahcW8gLnN7GuImI4niqyQbGhYlgFDm/40WVvjWfS1skRyaPDUkXorKFXl2MSw5yr/pN9Z31SyxuhbAQAA) **\]**

```kusto
let min_t = toscalar(demo_many_series1 | summarize min(TIMESTAMP));  
let max_t = toscalar(demo_many_series1 | summarize max(TIMESTAMP));  
demo_many_series1
| make-series reads=avg(DataRead) on TIMESTAMP in range(min_t, max_t, 1h) by Loc, Op, DB
| extend (rsquare, slope) = series_fit_line(reads)
| top 2 by slope asc
| project Loc, Op, DB, slope 
```

|   |   |   |   |   |
| --- | --- | --- | --- | --- |
|   | Loc | Op | DB | slope |
|   | Loc 15 | 37 | 1151 | -102743.910227889 |
|   | Loc 13 | 37 | 1249 | -86303.2334644601 |

En menos de dos minutos, Azure Data Explorer analizó aproximadamente 20 000 series temporales y detectó dos series temporales anormales en las que el recuento de lecturas bajó repentinamente.

Estas funcionalidades avanzadas, combinadas con el rápido rendimiento de Azure Data Explorer, proporcionan una solución única y eficaz para el análisis de series temporales.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre la [detección y previsión de anomalías de series temporales](/azure/data-explorer/anomaly-detection) en Azure Data Explorer.
* Obtenga información sobre las [capacidades de aprendizaje automático](/azure/data-explorer/machine-learning-clustering) en Azure Data Explorer.
---
title: complemento de activity_metrics-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe activity_metrics complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 8106d419f20dcacdec6386294a5b9ffb8d1bc8e2
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225911"
---
# <a name="activity_metrics-plugin"></a>complemento activity_metrics

Calcula métricas de actividad útiles (valores de recuento distintivos, recuento distinto de valores nuevos, tasa de retención y tasa de renovación) en función de la ventana período actual frente a la ventana período anterior (a diferencia de [activity_counts_metrics complemento](activity-counts-metrics-plugin.md) en el que cada período de tiempo se compara con *las* ventanas de tiempo anteriores).

```kusto
T | evaluate activity_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `activity_metrics(` *IdColumn* `,` *TimelineColumn* `,` [*Start* `,` *End* `,` ] *Window* [ `,` *DIM1* `,` *dim2* `,` ...]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: (opcional) escalar con el valor del período de inicio del análisis.
* *End*: (opcional) escalar con el valor del período de finalización del análisis.
* *Window*: escalar con el valor del período de la ventana de análisis. Puede ser un valor numérico, de fecha y hora o de marca de tiempo, o una cadena que sea uno de `week` / `month` / `year` , en cuyo caso todos los períodos serán [iniciodelasemana](startofweekfunction.md) / [startofmonth](startofmonthfunction.md) / [startofyear](startofyearfunction.md) en consecuencia. 
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de las métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene los valores de recuento distintivos, el recuento distintivo de valores nuevos, la tasa de retención y la tasa de renovación para cada período de la escala de tiempo y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|dcount_values|dcount_newvalues|retention_rate|churn_rate|DIM1|..|dim_n|
|---|---|---|---|---|--|--|--|--|--|--|
|tipo: a partir de *TimelineColumn*|long|long|double|double|..|..|..|

**Notas**

***Definición de la tasa de retención***

`Retention Rate`en un período se calcula de la siguiente manera:

    # of customers returned during the period
    / (divided by)
    # customers at the beginning of the period

donde `# of customers returned during the period` se define como:

    # of customers at end of period
    - (minus)
    # of new customers acquired during the period

`Retention Rate`puede variar de 0,0 a 1,0  
La puntuación más alta significa la mayor cantidad de usuarios devueltos.


***Definición de la tasa de renovación***

`Churn Rate`en un período se calcula de la siguiente manera:
    
    # of customers lost in the period
    / (divided by)
    # of customers at the beginning of the period

donde `# of customer lost in the period` se define como:

    # of customers at the beginning of the period
    - (minus)
    # of customers at the end of the period

`Churn Rate`puede variar entre 0,0 y 1,0, lo que significa que la mayor cantidad de usuarios no devuelve el servicio.

***Renovación frente a tasa de retención***

Derivado de la definición de `Churn Rate` y `Retention Rate` , lo siguiente siempre es true:

    [Retention rate] = 100.0% - [Churn Rate]


**Ejemplos**

### <a name="weekly-retention-rate-and-churn-rate"></a>Tasa de retención semanal y tasa de renovación

La consulta siguiente calcula la retención y la tasa de renovación de la ventana de semana a semana.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-02);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// 
| evaluate activity_metrics(['id'], _day, _start, _end, 7d)
| project _day, retention_rate, churn_rate
| render timechart 
```

|_day|retention_rate|churn_rate|
|---|---|---|
|2017-01-02 00:00:00.0000000|NaN|NaN|
|2017-01-09 00:00:00.0000000|0.179910044977511|0.820089955022489|
|2017-01-16 00:00:00.0000000|0.744374437443744|0.255625562556256|
|2017-01-23 00:00:00.0000000|0.612096774193548|0.387903225806452|
|2017-01-30 00:00:00.0000000|0.681141439205955|0.318858560794045|
|2017-02-06 00:00:00.0000000|0.278145695364238|0.721854304635762|
|2017-02-13 00:00:00.0000000|0.223172628304821|0.776827371695179|
|2017-02-20 00:00:00.0000000|0,38|0,62|
|2017-02-27 00:00:00.0000000|0.295519001701645|0.704480998298355|
|2017-03-06 00:00:00.0000000|0.280387770320656|0.719612229679344|
|2017-03-13 00:00:00.0000000|0.360628154795289|0.639371845204711|
|2017-03-20 00:00:00.0000000|0.288008028098344|0.711991971901656|
|2017-03-27 00:00:00.0000000|0.306134969325153|0.693865030674847|
|2017-04-03 00:00:00.0000000|0.356866537717602|0.643133462282398|
|2017-04-10 00:00:00.0000000|0.495098039215686|0.504901960784314|
|2017-04-17 00:00:00.0000000|0.198296836982968|0.801703163017032|
|2017-04-24 00:00:00.0000000|0.0618811881188119|0.938118811881188|
|2017-05-01 00:00:00.0000000|0.204657727593507|0.795342272406493|
|2017-05-08 00:00:00.0000000|0.517391304347826|0.482608695652174|
|2017-05-15 00:00:00.0000000|0.143667296786389|0.856332703213611|
|2017-05-22 00:00:00.0000000|0.199122325836533|0.800877674163467|
|2017-05-29 00:00:00.0000000|0.063468992248062|0.936531007751938|

:::image type="content" source="images/activity-metrics-plugin/activity-metrics-churn-and-retention.png" border="false" alt-text="Renovación y retención de métricas de actividad":::

### <a name="distinct-values-and-distinct-new-values"></a>Valores DISTINCT y valores ' New ' distintos 

La consulta siguiente calcula valores distintos y valores ' New ' (identificadores que no aparecían en la ventana de tiempo anterior) para la ventana de semana a semana.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-02);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// 
| evaluate activity_metrics(['id'], _day, _start, _end, 7d)
| project _day, dcount_values, dcount_newvalues
| render timechart 
```

|_day|dcount_values|dcount_newvalues|
|---|---|---|
|2017-01-02 00:00:00.0000000|630|630|
|2017-01-09 00:00:00.0000000|738|575|
|2017-01-16 00:00:00.0000000|1187|841|
|2017-01-23 00:00:00.0000000|1092|465|
|2017-01-30 00:00:00.0000000|1261|647|
|2017-02-06 00:00:00.0000000|1744|1043|
|2017-02-13 00:00:00.0000000|1563|432|
|2017-02-20 00:00:00.0000000|1406|818|
|2017-02-27 00:00:00.0000000|1956|1429|
|2017-03-06 00:00:00.0000000|1593|848|
|2017-03-13 00:00:00.0000000|1801|1423|
|2017-03-20 00:00:00.0000000|1710|1017|
|2017-03-27 00:00:00.0000000|1796|1516|
|2017-04-03 00:00:00.0000000|1381|1008|
|2017-04-10 00:00:00.0000000|1756|1162|
|2017-04-17 00:00:00.0000000|1831|1409|
|2017-04-24 00:00:00.0000000|1823|1164|
|2017-05-01 00:00:00.0000000|1811|1353|
|2017-05-08 00:00:00.0000000|1691|1246|
|2017-05-15 00:00:00.0000000|1812|1608|
|2017-05-22 00:00:00.0000000|1740|1017|
|2017-05-29 00:00:00.0000000|960|756|

:::image type="content" source="images/activity-metrics-plugin/activity-metrics-dcount-and-dcount-newvalues.png" border="false" alt-text="Métricas de actividad DCont y DCont nuevos valores":::

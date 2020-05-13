---
title: 'complemento de new_activity_metrics: Azure Explorador de datos'
description: En este artículo se describe new_activity_metrics complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 5e02c7ca2874a779cc5a626fd65522392439b491
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271594"
---
# <a name="new_activity_metrics-plugin"></a>complemento new_activity_metrics

Calcula métricas de actividad útiles (valores de recuento distintivos, recuento distinto de valores nuevos, tasa de retención y tasa de renovación) para el cohorte de `New Users` . Cada cohorte de `New Users` (todos los usuarios que se vieron en el primer período de tiempo) se compara con todos los cohortes anteriores. La comparación tiene en cuenta *todas las* ventanas de tiempo anteriores. Por ejemplo, en el registro de de = T2 y en = T3, el recuento distintivo de usuarios será todos los usuarios de T3 que no se vieron en T1 y T2. 
```kusto
T | evaluate new_activity_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `new_activity_metrics(` *IdColumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *Window* [ `,` *cohorte*] [ `,` *DIM1* `,` *dim2* `,` ...] [ `,` *lookback*]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: escalar con el valor del período de inicio del análisis.
* *End*: escalar con el valor del período de finalización del análisis.
* *Window*: escalar con el valor del período de la ventana de análisis. Puede ser un valor numérico, de fecha y hora o de marca de tiempo, o una cadena que sea uno de `week` / `month` / `year` , en cuyo caso todos los períodos serán [iniciodelasemana](startofweekfunction.md) / [startofmonth](startofmonthfunction.md) / [startofyear](startofyearfunction.md) en consecuencia. 
* *Cohorte*: (opcional) una constante escalar que indica un cohorte específico. Si no se proporciona, se calculan y devuelven todos los cohortes correspondientes a la ventana de tiempo de análisis.
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de las métricas de actividad.
* *Lookback*: (opcional) una expresión tabular con un conjunto de identificadores que pertenecen al período de "mirar hacia atrás"

**Devuelve**

Devuelve una tabla que tiene los valores de recuento distintivos, el recuento distintivo de valores nuevos, la tasa de retención y la tasa de renovación para cada combinación de períodos de escala de tiempo ' desde ' y ' hasta ' y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|from_TimelineColumn|to_TimelineColumn|dcount_new_values|dcount_retained_values|dcount_churn_values|retention_rate|churn_rate|DIM1|..|dim_n|
|---|---|---|---|---|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|same|long|long|double|double|double|..|..|..|

* `from_TimelineColumn`-el cohorte de nuevos usuarios. Las métricas de este registro hacen referencia a todos los usuarios que se vieron en primer lugar en este período. La decisión de la *primera* vista tiene en cuenta todos los períodos anteriores en el período de análisis. 
* `to_TimelineColumn`: el período en el que se compara. 
* `dcount_new_values`-el número de usuarios distintos en los `to_TimelineColumn` que no se han encontrado en *todos los* períodos anteriores a e incluidos `from_TimelineColumn` . 
* `dcount_retained_values`-de todos los usuarios nuevos, en primer lugar en `from_TimelineColumn` , el número de usuarios distintos que se han detectado en `to_TimelineCoumn` .
* `dcount_churn_values`-de todos los usuarios nuevos, en primer lugar en `from_TimelineColumn` , el número de usuarios distintos que *no* se han detectado en `to_TimelineCoumn` .
* `retention_rate`-el porcentaje de `dcount_retained_values` fuera del cohorte (los usuarios que se vieron en primer lugar en `from_TimelineColumn` ).
* `churn_rate`-el porcentaje de `dcount_churn_values` fuera del cohorte (los usuarios que se vieron en primer lugar en `from_TimelineColumn` ).

**Notas**

Para las definiciones de `Retention Rate` y `Churn Rate` , consulte la sección **notas** de la documentación del complemento de [activity_metrics](./activity-metrics-plugin.md) .


**Ejemplos**

En el siguiente conjunto de datos de ejemplo se muestran los usuarios que han aparecido en qué días. La tabla se generó en función de una `Users` tabla de origen, como se indica a continuación: 

```kusto
Users | summarize tostring(make_set(user)) by bin(Timestamp, 1d) | order by Timestamp asc;
```

|Timestamp|set_user|
|---|---|
|2019-11-01 00:00:00.0000000|[0, 2, 3, 4]|
|2019-11-02 00:00:00.0000000|[0, 1, 3, 4, 5]|
|2019-11-03 00:00:00.0000000|[0, 2, 4, 5]|
|2019-11-04 00:00:00.0000000|[0, 1, 2, 3]|
|2019-11-05 00:00:00.0000000|[0, 1, 2, 3, 4]|

La salida del complemento de la tabla original es la siguiente: 

```kusto
let StartDate = datetime(2019-11-01 00:00:00);
let EndDate = datetime(2019-11-07 00:00:00);
Users 
| evaluate new_activity_metrics(user, Timestamp, StartDate, EndDate-1tick, 1d) 
| where from_Timestamp < datetime(2019-11-03 00:00:00.0000000)
```

|R|from_Timestamp|to_Timestamp|dcount_new_values|dcount_retained_values|dcount_churn_values|retention_rate|churn_rate|
|---|---|---|---|---|---|---|---|
|1|2019-11-01 00:00:00.0000000|2019-11-01 00:00:00.0000000|4|4|0|1|0|
|2|2019-11-01 00:00:00.0000000|2019-11-02 00:00:00.0000000|2|3|1|0,75|0,25|
|3|2019-11-01 00:00:00.0000000|2019-11-03 00:00:00.0000000|1|3|1|0,75|0,25|
|4|2019-11-01 00:00:00.0000000|2019-11-04 00:00:00.0000000|1|3|1|0,75|0,25|
|5|2019-11-01 00:00:00.0000000|2019-11-05 00:00:00.0000000|1|4|0|1|0|
|6|2019-11-01 00:00:00.0000000|2019-11-06 00:00:00.0000000|0|0|4|0|1|
|7|2019-11-02 00:00:00.0000000|2019-11-02 00:00:00.0000000|2|2|0|1|0|
|8|2019-11-02 00:00:00.0000000|2019-11-03 00:00:00.0000000|0|1|1|0.5|0.5|
|9|2019-11-02 00:00:00.0000000|2019-11-04 00:00:00.0000000|0|1|1|0.5|0.5|
|10|2019-11-02 00:00:00.0000000|2019-11-05 00:00:00.0000000|0|1|1|0.5|0.5|
|11|2019-11-02 00:00:00.0000000|2019-11-06 00:00:00.0000000|0|0|2|0|1|

A continuación se muestra un análisis de algunos registros de la salida: 
* Registro `R=3` , `from_TimelineColumn`  =  `2019-11-01` , `to_TimelineColumn`  =  `2019-11-03` :
    * Los usuarios que se tienen en cuenta para este registro son los nuevos usuarios que se ven en 11/1. Dado que se trata del primer período, se trata de todos los usuarios de esa ubicación: [0, 2, 3, 4]
    * `dcount_new_values`: número de usuarios de 11/3 que no se han detectado en 11/1. Esto incluye un solo usuario: `5` . 
    * `dcount_retained_values`-fuera de todos los usuarios nuevos en 11/1, ¿cuántos se retuvieron hasta 11/3? Hay tres ( `[0,2,4]` ), mientras que `count_churn_values` es uno (usuario = `3` ). 
    * `retention_rate`= 0,75: los tres usuarios retenidos de los cuatro usuarios nuevos que se vieron por primera vez en 11/1. 

* Registro `R=9` , `from_TimelineColumn`  =  `2019-11-02` , `to_TimelineColumn`  =  `2019-11-04` :
    * Este registro se centra en los nuevos usuarios que se vieron por primera vez en 11/2: usuarios `1` y `5` . 
    * `dcount_new_values`: número de usuarios de 11/4 que no se han detectado a lo largo de todos los períodos `T0 .. from_Timestamp` . Es decir, los usuarios que se ven en 11/4 pero que no se vieron en 11/1 o 11/2, no hay ningún usuario de este tipo. 
    * `dcount_retained_values`-fuera de todos los usuarios nuevos en 11/2 ( `[1,5]` ), ¿cuántos se retuvieron hasta 11/4? Hay un usuario de este tipo ( `[1]` ), mientras que count_churn_values es uno (usuario `5` ). 
    * `retention_rate`es 0,5: el único usuario que se reservó en 11/4 de las dos nuevas en 11/2. 


### <a name="weekly-retention-rate-and-churn-rate-single-week"></a>Tasa de retención semanal y tasa de renovación (semana única)

La consulta siguiente calcula una tasa de retención y de renovación de la ventana de semana a semana para `New Users` cohorte (usuarios que llegaron en la primera semana).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
range Day from _start to _end  step 1d
| extend d = tolong((Day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Take only the first week cohort (last parameter)
| evaluate new_activity_metrics(['id'], Day, _start, _end, 7d, _start)
| project from_Day, to_Day, retention_rate, churn_rate
```

|from_Day|to_Day|retention_rate|churn_rate|
|---|---|---|---|
|2017-05-01 00:00:00.0000000|2017-05-01 00:00:00.0000000|1|0|
|2017-05-01 00:00:00.0000000|2017-05-08 00:00:00.0000000|0.544632768361582|0.455367231638418|
|2017-05-01 00:00:00.0000000|2017-05-15 00:00:00.0000000|0.031638418079096|0.968361581920904|
|2017-05-01 00:00:00.0000000|2017-05-22 00:00:00.0000000|0|1|
|2017-05-01 00:00:00.0000000|2017-05-29 00:00:00.0000000|0|1|


### <a name="weekly-retention-rate-and-churn-rate-complete-matrix"></a>Tasa de retención semanal y tasa de renovación (matriz completa)

La consulta siguiente calcula la retención y la tasa de renovación de la ventana de semana a semana para `New Users` cohorte. Si en el ejemplo anterior se calculan las estadísticas de una sola semana, la siguiente genera la tabla NxN para cada combinación de/a.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
range Day from _start to _end  step 1d
| extend d = tolong((Day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Last parameter is omitted - 
| evaluate new_activity_metrics(['id'], Day, _start, _end, 7d)
| project from_Day, to_Day, retention_rate, churn_rate
```

|from_Day|to_Day|retention_rate|churn_rate|
|---|---|---|---|
|2017-05-01 00:00:00.0000000|2017-05-01 00:00:00.0000000|1|0|
|2017-05-01 00:00:00.0000000|2017-05-08 00:00:00.0000000|0.190397350993377|0.809602649006622|
|2017-05-01 00:00:00.0000000|2017-05-15 00:00:00.0000000|0|1|
|2017-05-01 00:00:00.0000000|2017-05-22 00:00:00.0000000|0|1|
|2017-05-01 00:00:00.0000000|2017-05-29 00:00:00.0000000|0|1|
|2017-05-08 00:00:00.0000000|2017-05-08 00:00:00.0000000|1|0|
|2017-05-08 00:00:00.0000000|2017-05-15 00:00:00.0000000|0.405263157894737|0.594736842105263|
|2017-05-08 00:00:00.0000000|2017-05-22 00:00:00.0000000|0.227631578947368|0.772368421052632|
|2017-05-08 00:00:00.0000000|2017-05-29 00:00:00.0000000|0|1|
|2017-05-15 00:00:00.0000000|2017-05-15 00:00:00.0000000|1|0|
|2017-05-15 00:00:00.0000000|2017-05-22 00:00:00.0000000|0.785488958990536|0.214511041009464|
|2017-05-15 00:00:00.0000000|2017-05-29 00:00:00.0000000|0.237644584647739|0.762355415352261|
|2017-05-22 00:00:00.0000000|2017-05-22 00:00:00.0000000|1|0|
|2017-05-22 00:00:00.0000000|2017-05-29 00:00:00.0000000|0.621835443037975|0.378164556962025|
|2017-05-29 00:00:00.0000000|2017-05-29 00:00:00.0000000|1|0|


### <a name="weekly-retention-rate-with-lookback-period"></a>Tasa de retención semanal con período de lookback

La consulta siguiente calcula la tasa de retención de `New Users` cohorte cuando se toma en consideración el `lookback` período: una consulta tabular con un conjunto de identificadores que se usan para definir el `New Users` cohorte (todos los identificadores que no aparecen en este conjunto `New Users` ). La consulta examina el comportamiento de retención de `New Users` durante el período de análisis.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _lookback = datetime(2017-02-01);
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
let _data = range Day from _lookback to _end  step 1d
| extend d = tolong((Day - _lookback)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000;
//
let lookback_data = _data | where Day < _start | project Day, id;
_data
| evaluate new_activity_metrics(id, Day, _start, _end, 7d, _start, lookback_data)
| project from_Day, to_Day, retention_rate
```

|from_Day|to_Day|retention_rate|
|---|---|---|
|2017-05-01 00:00:00.0000000|2017-05-01 00:00:00.0000000|1|
|2017-05-01 00:00:00.0000000|2017-05-08 00:00:00.0000000|0.404081632653061|
|2017-05-01 00:00:00.0000000|2017-05-15 00:00:00.0000000|0.257142857142857|
|2017-05-01 00:00:00.0000000|2017-05-22 00:00:00.0000000|0.296326530612245|
|2017-05-01 00:00:00.0000000|2017-05-29 00:00:00.0000000|0.0587755102040816|

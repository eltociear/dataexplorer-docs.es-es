---
title: new_activity_metrics plugin - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe new_activity_metrics complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 0aad1c91fec6855030544596a08818b80bbf3d18
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512213"
---
# <a name="new_activity_metrics-plugin"></a>complemento new_activity_metrics

Calcula métricas de actividad útiles (valores de recuento distintos, recuento distinto de `New Users`nuevos valores, tasa de retención y tasa de rotación) para la cohorte de . Cada cohorte de `New Users` (todos los usuarios que fueron vistos en la ventana de tiempo) se compara con todas las cohortes anteriores. La comparación tiene en cuenta *todas las* ventanas de tiempo anteriores. Por ejemplo, en el registro de from-T2 y to-T3, el recuento distinto de usuarios serán todos los usuarios en T3 que no se vieron en T1 y T2. 
```kusto
T | evaluate new_activity_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `,` *End* `,` *Window* `,` `,` `,` `,` *IdColumn* `,` `,` *Start* *dim2* *TimelineColumn* *dim1* *Cohort*IdColumn TimelineColumn Ventana de inicio de columna [ Cohorte ] [ dim1 dim2 ...] `new_activity_metrics(` [`,` *Mirar hacia atrás*]`)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColumn*: el nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: el nombre de la columna que representa la línea de tiempo.
* *Inicio*: Escalar con el valor del período de inicio del análisis.
* *Final*: Escalar con el valor del período final del análisis.
* *Ventana*: Escalar con el valor del período de la ventana de análisis. Puede ser un valor numérico/datetime/timestamp, o una `week` / `month` / `year`cadena que sea uno de , en cuyo caso todos los períodos serán [el inicio del](startofweekfunction.md)/[mes](startofmonthfunction.md)/de la semana en[consecuencia.](startofyearfunction.md) 
* *Cohorte*: (opcional) una constante escalar que indica una cohorte específica. Si no se proporciona, se calculan y devuelven todas las cohortes correspondientes a la ventana de tiempo de análisis.
* *dim1*, *dim2*, ...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de métricas de actividad.
* *Lookback*: (opcional) una expresión tabular con un conjunto de iDE que pertenecen al período 'mirar hacia atrás'

**Devuelve**

Devuelve una tabla que tiene los valores de recuento distintos, recuento distinto de nuevos valores, tasa de retención y tasa de abandono para cada combinación de períodos de escala de tiempo "de" y "a" y para cada combinación de dimensiones existentes.

El esquema de la tabla de salida es:

|from_TimelineColumn|to_TimelineColumn|dcount_new_values|dcount_retained_values|dcount_churn_values|retention_rate|churn_rate|dim1|..|dim_n|
|---|---|---|---|---|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|same|long|long|double|double|double|..|..|..|

* `from_TimelineColumn`- la cohorte de nuevos usuarios. Las métricas de este registro se refieren a todos los usuarios que se vieron por primera vez en este período. La decisión sobre la *primera vista* tiene en cuenta todos los períodos anteriores en el período de análisis. 
* `to_TimelineColumn`- el período que se compara con. 
* `dcount_new_values`- el número de `to_TimelineColumn` usuarios distintos en los que `from_TimelineColumn`no se vieron en todos *los* períodos anteriores e incluidos . 
* `dcount_retained_values`- de todos los nuevos usuarios, vistos por primera vez en `from_TimelineColumn`, el número de usuarios distintos que se vieron en `to_TimelineCoumn`.
* `dcount_churn_values`- de todos los nuevos usuarios, vistos por primera vez en `from_TimelineColumn`, el número de usuarios distintos que *no* se vieron en `to_TimelineCoumn`.
* `retention_rate`- el `dcount_retained_values` porcentaje de fuera de la `from_TimelineColumn`cohorte (usuarios vistos por primera vez en ).
* `churn_rate`- el `dcount_churn_values` porcentaje de fuera de la `from_TimelineColumn`cohorte (usuarios vistos por primera vez en ).

**Notas**

Para obtener `Retention Rate` definiciones de y `Churn Rate` - consulte la sección **Notas** en activity_metrics documentación [del complemento.](./activity-metrics-plugin.md)


**Ejemplos**

El siguiente conjunto de datos de ejemplo muestra qué usuarios han visto en qué días. La tabla se generó `Users` basándose en una tabla de origen, como se indica a continuación: 

```kusto
Users | summarize tostring(make_set(user)) by bin(Timestamp, 1d) | order by Timestamp asc;
```

|Timestamp|set_user|
|---|---|
|2019-11-01 00:00:00.0000000|[0,2,3,4]|
|2019-11-02 00:00:00.0000000|[0,1,3,4,5]|
|2019-11-03 00:00:00.0000000|[0,2,4,5]|
|2019-11-04 00:00:00.0000000|[0,1,2,3]|
|2019-11-05 00:00:00.0000000|[0,1,2,3,4]|

La salida del plugin para la tabla original es la siguiente: 

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
* Registro `R=3` `from_TimelineColumn`  =  `2019-11-01`, `to_TimelineColumn`  = , `2019-11-03`:
    * Los usuarios considerados para este registro son todos los nuevos usuarios vistos en 11/1. Dado que este es el primer período, estos son todos los usuarios en esa ubicación – [0,2,3,4]
    * `dcount_new_values`– el número de usuarios en 11/3 que no fueron vistos en 11/1. Esto incluye un `5`solo usuario – . 
    * `dcount_retained_values`– de todos los nuevos usuarios en 11/1, ¿cuántos se retuvieron hasta el 11/3? Hay tres`[0,2,4]`( `count_churn_values` ), mientras`3`que es uno (usuario ). 
    * `retention_rate`0,75 – los tres usuarios retenidos de los cuatro nuevos usuarios que fueron vistos por primera vez en 11/1. 

* Registro `R=9` `from_TimelineColumn`  =  `2019-11-02`, `to_TimelineColumn`  = , `2019-11-04`:
    * Este registro se centra en los nuevos usuarios que fueron `1` `5`vistos por primera vez en 11x2 – usuarios y . 
    * `dcount_new_values`– el número de usuarios en 11/4 que `T0 .. from_Timestamp`no fueron vistos en todos los períodos. Es decir, los usuarios que se ven en 11/4 pero que no fueron vistos en 11/1 o 11/2 – no hay tales usuarios. 
    * `dcount_retained_values`– de todos los nuevos usuarios`[1,5]`en 11/2 ( ), ¿cuántos se retuvieron hasta el 11-4? Hay uno de estos`[1]`usuarios ( ), `5`mientras que count_churn_values es uno (usuario ). 
    * `retention_rate`es 0.5 – el usuario único que se retuvo en 11x4 de los dos nuevos en 11/2. 


### <a name="weekly-retention-rate-and-churn-rate-single-week"></a>Tasa de retención semanal y tasa de abandono (semana única)

La siguiente consulta calcula una tasa de retención y `New Users` abandono para la ventana semana a semana para la cohorte (usuarios que llegaron la primera semana).

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


### <a name="weekly-retention-rate-and-churn-rate-complete-matrix"></a>Tasa de retención semanal y tasa de abandono (matriz completa)

La siguiente consulta calcula la retención y la tasa `New Users` de abandono para la ventana semana a semana para la cohorte. Si el ejemplo anterior calculó las estadísticas para una sola semana , la siguiente produce la tabla NxN para cada combinación de/hasta.

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


### <a name="weekly-retention-rate-with-lookback-period"></a>Tasa de retención semanal con periodo de retención

La siguiente consulta calcula la `New Users` tasa de `lookback` retención de la cohorte al tener en `New Users` cuenta el período: una consulta tabular con `New Users`conjunto de identificadores que se utilizan para definir la cohorte (todos los identificadores que no aparecen en este conjunto son ). La consulta examina el comportamiento `New Users` de retención del período de análisis durante el período de análisis.

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

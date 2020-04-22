---
title: Partición y composición de resultados intermedios de agregaciones - Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describe la creación de particiones y la composición de resultados intermedios de agregaciones en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: kusto/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 1392105e4b5d99f2273b686ed74a161750ee6f8c
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81662962"
---
# <a name="partitioning-and-composing-intermediate-results-of-aggregations"></a>Partición y composición de resultados intermedios de agregaciones

Supongamos que desea calcular el recuento de usuarios distintos en los últimos siete días, todos los días. Una forma de hacerlo sería `summarize dcount(user)` ejecutaruna vez al día con un intervalo filtrado a los últimos siete días. Esto es ineficiente porque cada vez que se ejecuta el cálculo hay una superposición de seis días con el cálculo anterior. Otra opción es calcular algún agregado para cada día y, a continuación, combinar estos agregados de una manera eficaz. Esta opción requiere que "recuerde" los últimos seis resultados, pero es mucho más eficiente.

Particionar consultas como esa sería fácil para agregados simples, como count() y sum(). Sin embargo, también es posible realizarlos para agregados más complejos, como dcount() y percentiles(). En este tema se explica cómo Kusto admite estos cálculos.

Los siguientes ejemplos muestran cómo utilizar hll/tdigest y demuestran que el uso de estos en algunos escenarios puede ser mucho más performante que de otras maneras:

> [!NOTE]
> En algunos casos, los objetos `hll` dinámicos generados por las funciones de `tdigest` agregado o pueden ser grandes y superar la propiedad MaxValueSize predeterminada en la directiva de codificación. Si es así, el objeto se ingerirá como null.
Por ejemplo, al conservar `hll` la salida de la función con el nivel de precisión 4, el tamaño del objeto hll supera el valor predeterminado MaxValueSize que es 1 MB.

```kusto
range x from 1 to 1000000 step 1
| summarize hll(x,4)
| project sizeInMb = estimate_data_size(hll_x) / pow(1024,2)
```

|sizeInMb|
|---|
|1.0000524520874|

Ingeridos esto en una tabla antes de aplicar este tipo de directiva ingerirá null:

```kusto
.set-or-append MyTable <| range x from 1 to 1000000 step 1
| summarize hll(x,4)
```

```kusto
MyTable
| project isempty(hll_x)
```

| Column1 |
|---------|
| 1       |

Para evitar esto, utilice el `bigobject`tipo de directiva de codificación especial , que reemplaza MaxValueSize a 2MB de la siguiente manera:

```kusto
.alter column MyTable.hll_x policy encoding type='bigobject'
```



Así que ingiriendo un valor ahora a la misma tabla anterior:

```kusto
.set-or-append MyTable <| range x from 1 to 1000000 step 1
| summarize hll(x,4)
```
ingesta del segundo valor con éxito: 

```kusto
MyTable
| project isempty(hll_x)
```

|Column1|
|---|
|1|
|0|


**Ejemplo**

Supongamos que hay una tabla, PageViewsHllTDigest, que tiene los valores hll sobre Pages vistos en cada hora. El objetivo es obtener estos valores, `12h`pero binned to , para combinar los valores hll `12h`utilizando la función de agregado hll_merge() por marca de tiempo binned a . A continuación, llame a la función dcount_hll para obtener el valor dcount final:

```kusto
PageViewsHllTDigest
| summarize merged_hll = hll_merge(hllPage) by bin(Timestamp, 12h)
| project Timestamp , dcount_hll(merged_hll)
```

|Timestamp|dcount_hll_merged_hll|
|---|---|
|2016-05-01 12:00:00.0000000|20056275|
|2016-05-02 00:00:00.0000000|38797623|
|2016-05-02 12:00:00.0000000|39316056|
|2016-05-03 00:00:00.0000000|13685621|

O incluso para la `1d`marca de tiempo binned para :

```kusto
PageViewsHllTDigest
| summarize merged_hll = hll_merge(hllPage) by bin(Timestamp, 1d)
| project Timestamp , dcount_hll(merged_hll)
```

|Timestamp|dcount_hll_merged_hll|
|---|---|
|2016-05-01 00:00:00.0000000|20056275|
|2016-05-02 00:00:00.0000000|64135183|
|2016-05-03 00:00:00.0000000|13685621|

 Lo mismo se puede hacer sobre los valores de tdigest, que representan bytes entregados en cada hora:

```kusto
PageViewsHllTDigest
| summarize merged_tdigests = merge_tdigests(tdigestBytesDel) by bin(Timestamp, 12h)
| project Timestamp , percentile_tdigest(merged_tdigests, 95, typeof(long))
```

|Timestamp|percentile_tdigest_merged_tdigests|
|---|---|
|2016-05-01 12:00:00.0000000|170200|
|2016-05-02 00:00:00.0000000|152975|
|2016-05-02 12:00:00.0000000|181315|
|2016-05-03 00:00:00.0000000|146817|
 
**Ejemplo**

Los límites de Kusto se alcanzan con conjuntos de datos que son demasiado grandes, donde [`percentile()`](percentiles-aggfunction.md) debe [`dcount()`](dcount-aggfunction.md) ejecutar consultas periódicas sobre el conjunto de datos, pero ejecutar las consultas regulares para calcular o sobre estos conjuntos de datos grandes.

::: zone pivot="azuredataexplorer"

Para resolver este problema, los datos recién agregados se pueden agregar [`hll()`](hll-aggfunction.md) a una tabla temporal [`tdigest()`](tdigest-aggfunction.md) como valores hll o [`set/append`](../management/data-ingestion/index.md) [`update policy`](../management/updatepolicy.md)tdigest utilizando cuando la operación requerida es dcount o cuando la operación requerida es percentil usando o , En este caso, los resultados intermedios de dcount o tdigest se guardan en otro conjunto de datos, que debe ser menor que el grande de destino.

::: zone-end

::: zone pivot="azuremonitor"

Para resolver este problema, los datos recién agregados se pueden agregar [`hll()`](hll-aggfunction.md) a una tabla temporal como valores hll o tdigest utilizando cuando la operación requerida es dcount, En este caso, los resultados intermedios de dcount se guardan en otro conjunto de datos, que debe ser menor que el gran destino.

::: zone-end

Cuando necesite obtener los resultados finales de estos valores, las consultas [`hll-merge()`](hll-merge-aggfunction.md) / [`tdigest_merge()`](tdigest-merge-aggfunction.md)pueden usar fusiones hll/tdigest: , Luego, después de obtener los valores combinados, [`percentile_tdigest()`](percentile-tdigestfunction.md)  /  [`dcount_hll()`](dcount-hllfunction.md) se pueden invocar en estos valores combinados para obtener el resultado final de dcount o percentiles.

Suponiendo que hay una tabla, PageViews, en la que los datos se ingenian diariamente, todos los días en los que desea calcular el recuento distinto de páginas vistas por minuite más tarde que la fecha - datetime(2016-05-01 18:00:00.0000000).

Ejecute la siguiente consulta:

```kusto
PageViews   
| where Timestamp > datetime(2016-05-01 18:00:00.0000000)
| summarize percentile(BytesDelivered, 90), dcount(Page,2) by bin(Timestamp, 1d)
```

|Timestamp|percentile_BytesDelivered_90|dcount_Page|
|---|---|---|
|2016-05-01 00:00:00.0000000|83634|20056275|
|2016-05-02 00:00:00.0000000|82770|64135183|
|2016-05-03 00:00:00.0000000|72920|13685621|

Esta consulta agregará todos los valores cada vez que ejecute esta consulta (por ejemplo, si desea ejecutarla muchas veces al día).

Si guarda los valores hll y tdigest (que son los resultados intermedios de dcount y percentile) en una tabla temporal, PageViewsHllTDigest, mediante una directiva de actualización o comandos set/append, solo puede combinar los valores y, a continuación, utilizar dcount_hll/percentile_tdigest mediante la siguiente consulta:

```kusto
PageViewsHllTDigest
| summarize  percentile_tdigest(merge_tdigests(tdigestBytesDel), 90), dcount_hll(hll_merge(hllPage)) by bin(Timestamp, 1d)
```

|Timestamp|percentile_tdigest_merge_tdigests_tdigestBytesDel|dcount_hll_hll_merge_hllPage|
|---|---|---|
|2016-05-01 00:00:00.0000000|84224|20056275|
|2016-05-02 00:00:00.0000000|83486|64135183|
|2016-05-03 00:00:00.0000000|72247|13685621|

Esto debería ser más performante y la consulta se ejecuta sobre una tabla más pequeña (en este ejemplo, la primera consulta se ejecuta sobre 215M registros mientras que la segunda se ejecuta solo sobre 32 registros).

**Ejemplo**

La consulta de retención.
Suponiendo que tiene una tabla que resume cuándo se vio cada página de Wikipedia (el tamaño de la muestra es 10M), y desea encontrar para cada fecha1 date2 el porcentaje de páginas revisadas tanto en date1 como en date2 en relación con las páginas vistas en date1 (date1 < date2).
  
La forma trivial utiliza los operadores join y summarize :

```kusto
// Get the total pages viewed each day
let totalPagesPerDay = PageViewsSample
| summarize by Page, Day = startofday(Timestamp)
| summarize count() by Day;
// Join the table to itself to get a grid where 
// each row shows foreach page1, in which two dates
// it was viewed.
// Then count the pages between each two dates to
// get how many pages were viewed between date1 and date2.
PageViewsSample
| summarize by Page, Day1 = startofday(Timestamp)
| join kind = inner
(
    PageViewsSample
    | summarize by Page, Day2 = startofday(Timestamp)
)
on Page
| where Day2 > Day1
| summarize count() by Day1, Day2
| join kind = inner
    totalPagesPerDay
on $left.Day1 == $right.Day
| project Day1, Day2, Percentage = count_*100.0/count_1
```

|Día 1|Día 2|Porcentaje|
|---|---|---|
|2016-05-01 00:00:00.0000000|2016-05-02 00:00:00.0000000|34.0645725975255|
|2016-05-01 00:00:00.0000000|2016-05-03 00:00:00.0000000|16.618368960101|
|2016-05-02 00:00:00.0000000|2016-05-03 00:00:00.0000000|14.6291376489636|

 
La consulta anterior tardó 18 segundos.

Cuando se utilizan [`hll()`](hll-aggfunction.md) [`hll_merge()`](hll-merge-aggfunction.md) las [`dcount_hll()`](dcount-hllfunction.md)funciones de , y , la consulta equivalente terminará después de 1,3 segundos y muestra que las funciones hll aceleran la consulta anterior en 14 veces:

```kusto
let Stats=PageViewsSample | summarize pagehll=hll(Page, 2) by day=startofday(Timestamp); // saving the hll values (intermediate results of the dcount values)
let day0=toscalar(Stats | summarize min(day)); // finding the min date over all dates.
let dayn=toscalar(Stats | summarize max(day)); // finding the max date over all dates.
let daycount=tolong((dayn-day0)/1d); // finding the range between max and min
Stats
| project idx=tolong((day-day0)/1d), day, pagehll
| mv-expand pidx=range(0, daycount) to typeof(long)
// Extend the column to get the dcount value from hll'ed values for each date (same as totalPagesPerDay from the above query)
| extend key1=iff(idx < pidx, idx, pidx), key2=iff(idx < pidx, pidx, idx), pages=dcount_hll(pagehll)
// For each two dates, merge the hll'ed values to get the total dcount over each two dates, 
// This helps to get the pages viewed in both date1 and date2 (see the description below about the intersection_size)
| summarize (day1, pages1)=arg_min(day, pages), (day2, pages2)=arg_max(day, pages), union_size=dcount_hll(hll_merge(pagehll)) by key1, key2
| where day2 > day1
// To get pages viewed in date1 and also date2, look at the merged dcount of date1 and date2, subtract it from pages of date1 + pages on date2.
| project pages1, day1,day2, intersection_size=(pages1 + pages2 - union_size)
| project day1, day2, Percentage = intersection_size*100.0 / pages1
```

|día1|día2|Porcentaje|
|---|---|---|
|2016-05-01 00:00:00.0000000|2016-05-02 00:00:00.0000000|33.2298494510578|
|2016-05-01 00:00:00.0000000|2016-05-03 00:00:00.0000000|16.9773830213667|
|2016-05-02 00:00:00.0000000|2016-05-03 00:00:00.0000000|14.5160020350006|

> [!NOTE] 
> Los resultados de las consultas no son 100% precisos debido al error de las funciones hll. (ver [`dcount()`](dcount-aggfunction.md) para más información sobre los errores).

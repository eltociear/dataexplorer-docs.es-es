---
title: 'Kusto Partition & Compose los resultados de la agregación intermedia: Azure Explorador de datos'
description: En este artículo se describe la creación de particiones y la composición de resultados intermedios de agregaciones en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 3f1371fe298b2d0e066fc3a278cc3b560050416c
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011591"
---
# <a name="partitioning-and-composing-intermediate-results-of-aggregations"></a>Creación de particiones y composición de resultados intermedios de agregaciones

Supongamos que desea calcular el recuento de usuarios distintos cada día durante los últimos siete días. Puede ejecutar una `summarize dcount(user)` vez al día con un intervalo filtrado hasta los últimos siete días. Este método es ineficaz, ya que cada vez que se ejecuta el cálculo, hay una superposición de seis días con el cálculo anterior. También puede calcular un agregado para cada día y, a continuación, combinar estos agregados. Este método requiere que "Recuerde" los seis últimos resultados, pero es mucho más eficaz.

La creación de particiones de consultas como se describe es fácil para los agregados simples, como `count()` y `sum()` . También puede ser útil para los agregados complejos, como `dcount()` y `percentiles()` . En este tema se explica cómo Kusto admite estos cálculos.

En los siguientes ejemplos se muestra cómo usar `hll` / `tdigest` y demostrar que el uso de estos comandos tiene un alto rendimiento en algunos escenarios:

> [!NOTE]
> En algunos casos, los objetos dinámicos generados por `hll` o las `tdigest` funciones de agregado pueden ser grandes y superar la propiedad MaxValueSize predeterminada en la Directiva de codificación. Si es así, el objeto se ingerirá como null.
Por ejemplo, al conservar la salida de la `hll` función con el nivel de precisión 4, el tamaño del `hll` objeto supera el valor predeterminado de MaxValueSize, que es de 1 MB.

```kusto
range x from 1 to 1000000 step 1
| summarize hll(x,4)
| project sizeInMb = estimate_data_size(hll_x) / pow(1024,2)
```

|sizeInMb|
|---|
|1,0000524520874|

La ingesta de este objeto en una tabla antes de aplicar este tipo de directiva inscribirá NULL:

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

Para evitar la ingesta de valores NULL, use el tipo de directiva de codificación especial `bigobject` , que reemplaza `MaxValueSize` a 2 MB de la siguiente manera:

```kusto
.alter column MyTable.hll_x policy encoding type='bigobject'
```

Ingerir un valor ahora a la misma tabla anterior:

```kusto
.set-or-append MyTable <| range x from 1 to 1000000 step 1
| summarize hll(x,4)
```

ingeri el segundo valor correctamente: 

```kusto
MyTable
| project isempty(hll_x)
```

|Column1|
|---|
|1|
|0|


**Ejemplo**

Hay una tabla, `PageViewsHllTDigest` , que contiene `hll` los valores de las páginas que se ven en cada hora. Quiere que estos valores discretizann `12h` . Combine los `hll` valores mediante la `hll_merge()` función de agregado, con la marca de tiempo discretizan en `12h` . Utilice la función `dcount_hll` para devolver el `dcount` valor final:

```kusto
PageViewsHllTDigest
| summarize merged_hll = hll_merge(hllPage) by bin(Timestamp, 12h)
| project Timestamp , dcount_hll(merged_hll)
```

|Timestamp|`dcount_hll_merged_hll`|
|---|---|
|2016-05-01 12:00:00.0000000|20056275|
|2016-05-02 00:00:00.0000000|38797623|
|2016-05-02 12:00:00.0000000|39316056|
|2016-05-03 00:00:00.0000000|13685621|

A la marca de tiempo de bin para `1d` :

```kusto
PageViewsHllTDigest
| summarize merged_hll = hll_merge(hllPage) by bin(Timestamp, 1d)
| project Timestamp , dcount_hll(merged_hll)
```

|Timestamp|`dcount_hll_merged_hll`|
|---|---|
|2016-05-01 00:00:00.0000000|20056275|
|2016-05-02 00:00:00.0000000|64135183|
|2016-05-03 00:00:00.0000000|13685621|

La misma consulta se puede realizar en los valores de `tdigest` , que representan el `BytesDelivered` en cada hora:

```kusto
PageViewsHllTDigest
| summarize merged_tdigests = merge_tdigests(tdigestBytesDel) by bin(Timestamp, 12h)
| project Timestamp , percentile_tdigest(merged_tdigests, 95, typeof(long))
```

|Timestamp|`percentile_tdigest_merged_tdigests`|
|---|---|
|2016-05-01 12:00:00.0000000|170200|
|2016-05-02 00:00:00.0000000|152975|
|2016-05-02 12:00:00.0000000|181315|
|2016-05-03 00:00:00.0000000|146817|
 
**Ejemplo**

Se alcanzan los límites de Kusto con los conjuntos de valores demasiado grandes, en los que es necesario ejecutar consultas periódicas en el conjunto de DataSet, pero ejecutar las consultas regulares para calcular [`percentile()`](percentiles-aggfunction.md) o [`dcount()`](dcount-aggfunction.md) sobre grandes conjuntos de valores.

::: zone pivot="azuredataexplorer"

Para solucionar este problema, los datos recién agregados se pueden agregar a una tabla temporal como `hll` `tdigest` valores o mediante [`hll()`](hll-aggfunction.md) cuando la operación necesaria sea `dcount` o [`tdigest()`](tdigest-aggfunction.md) cuando la operación necesaria sea percentil con [`set/append`](../../ingest-data-overview.md) o [`update policy`](../management/updatepolicy.md) . En este caso, los resultados intermedios de `dcount` o `tdigest` se guardan en otro conjunto de resultados, que debe ser menor que el destino grande.

::: zone-end

::: zone pivot="azuremonitor"

Para solucionar este problema, los datos recién agregados se pueden agregar a una tabla temporal como `hll` `tdigest` valores o mediante [`hll()`](hll-aggfunction.md) cuando la operación necesaria sea `dcount` . En este caso, los resultados intermedios de `dcount` se guardan en otro conjunto de resultados, que debe ser menor que el destino grande.

::: zone-end

Cuando necesite obtener los resultados finales de estos valores, las consultas pueden usar `hll` / `tdigest` fusiones: [`hll-merge()`](hll-merge-aggfunction.md) / [`tdigest_merge()`](tdigest-merge-aggfunction.md) . Después, después de obtener los valores combinados, [`percentile_tdigest()`](percentile-tdigestfunction.md)  /  [`dcount_hll()`](dcount-hllfunction.md) se puede invocar en estos valores combinados para obtener el resultado final de `dcount` o percentiles.

Suponiendo que hay una tabla, PageViews, en la que los datos se introducen diariamente, cada día en el que desea calcular el recuento distintivo de páginas vistas por minuto después de Date = DateTime (2016-05-01 18:00:00.0000000).

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

Esta consulta agregará todos los valores cada vez que ejecute esta consulta (por ejemplo, si desea ejecutarlo muchas veces al día).

Si guarda los `hll` valores y `tdigest` (que son los resultados intermedios de `dcount` y el percentil) en una tabla temporal, `PageViewsHllTDigest` , mediante una directiva de actualización o los comandos SET/append, solo podrá combinar los valores y, a continuación, usar `dcount_hll` / `percentile_tdigest` la siguiente consulta:

```kusto
PageViewsHllTDigest
| summarize  percentile_tdigest(merge_tdigests(tdigestBytesDel), 90), dcount_hll(hll_merge(hllPage)) by bin(Timestamp, 1d)
```

|Timestamp|`percentile_tdigest_merge_tdigests_tdigestBytesDel`|`dcount_hll_hll_merge_hllPage`|
|---|---|---|
|2016-05-01 00:00:00.0000000|84224|20056275|
|2016-05-02 00:00:00.0000000|83486|64135183|
|2016-05-03 00:00:00.0000000|72247|13685621|

Esta consulta debe ser más eficaz, ya que se ejecuta en una tabla más pequeña. En este ejemplo, la primera consulta se ejecuta más de ~ 215M registros, mientras que la segunda se ejecuta solo en 32 registros:

**Ejemplo**

Consulta de retención.
Supongamos que tiene una tabla que resume Cuándo se vio cada página de Wikipedia (el tamaño de la muestra es 10 millones de páginas) y desea buscar por cada fecha1 fecha2 el porcentaje de páginas revisadas en fecha1 y fecha2 en relación con las páginas vistas en fecha1 (fecha1 < fecha2).
  
La forma trivial es usar operadores de combinación y resumir:

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

 
La consulta anterior tardó aproximadamente 18 segundos.

Cuando se usan las [`hll()`](hll-aggfunction.md) [`hll_merge()`](hll-merge-aggfunction.md) funciones, y [`dcount_hll()`](dcount-hllfunction.md) , la consulta equivalente terminará después de ~ 1,3 segundos y mostrará que las `hll` funciones aceleran la consulta anterior en ~ 14 veces:

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

|DAY1|DAY2|Porcentaje|
|---|---|---|
|2016-05-01 00:00:00.0000000|2016-05-02 00:00:00.0000000|33.2298494510578|
|2016-05-01 00:00:00.0000000|2016-05-03 00:00:00.0000000|16.9773830213667|
|2016-05-02 00:00:00.0000000|2016-05-03 00:00:00.0000000|14.5160020350006|

> [!NOTE] 
> Los resultados de las consultas no son exactamente del 100% debido al error de las `hll` funciones. Para obtener más información acerca de los errores, vea [`dcount()`](dcount-aggfunction.md) .

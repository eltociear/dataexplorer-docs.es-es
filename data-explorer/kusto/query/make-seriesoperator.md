---
title: 'operador make-series: Azure Explorador de datos'
description: En este artículo se describe el operador make-series en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/16/2020
ms.openlocfilehash: b4df3d7ea6df9eaec2e71fd3dddd60a1b23a02bd
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271441"
---
# <a name="make-series-operator"></a>Operador make-series

Crea una serie de valores agregados especificados a lo largo del eje especificado. 

```kusto
T | make-series sum(amount) default=0, avg(price) default=0 on timestamp from datetime(2016-01-01) to datetime(2016-01-10) step 1d by fruit, supplier
```

**Sintaxis**

*T* `| make-series` [*MakeSeriesParamters*] [*columna* `=` ] *agregación* [ `default` `=` *DefaultValue*] [ `,` ...] `on` *AxisColumn* [ `from` *Inicio*] [ `to` *End*] `step` *Step* [ `by` [*columna* `=` ] *GroupExpression* [ `,` ...]]

**Argumentos**

* *Column* : nombre opcional para una columna de resultados. El valor predeterminado es un nombre derivado de la expresión.
* *DefaultValue:* Valor predeterminado que se usará en lugar de los valores ausentes. Si no hay ninguna fila con valores específicos de *AxisColumn* y *GroupExpression* , en los resultados se asignará el elemento correspondiente de la matriz con un valor *DefaultValue*. Si `default =` *DefaultValue* se omite, se supone que es 0. 
* *Agregación:* Una llamada a una [función de agregación](make-seriesoperator.md#list-of-aggregation-functions) como `count()` o `avg()` , con los nombres de columna como argumentos. Consulte la [lista de funciones de agregación](make-seriesoperator.md#list-of-aggregation-functions). Tenga en cuenta que solo se pueden utilizar con el operador las funciones de agregación que devuelven resultados numéricos `make-series` .
* *AxisColumn:* Columna en la que se ordenará la serie. Puede considerarse como una escala de tiempo, pero además de `datetime` los tipos numéricos que se aceptan.
* *Inicio*: (opcional) se generará el valor de límite inferior de *AxisColumn* para cada serie. *Start*, *End* y *Step* se utilizan para generar una matriz de valores *AxisColumn* dentro de un intervalo determinado y usando el *paso*especificado. Todos los valores de *agregación* se ordenan respectivamente a esta matriz. Esta matriz *AxisColumn* también es la última columna de salida en la salida con el mismo nombre que *AxisColumn*. Si no se especifica un valor de *Inicio* , el inicio es el primer bin (paso) que tiene los datos en cada serie.
* *End*: (opcional) el valor límite alto (no inclusivo) del *AxisColumn*, el último índice de la serie temporal es menor que este valor (y será *Start* más múltiplo entero del *paso* menor que *End*). Si no se proporciona el valor *final* , será el límite superior de la última ubicación (paso) que tiene datos por cada serie.
* *Step*: la diferencia entre dos elementos consecutivos de la matriz *AxisColumn* (es decir, el tamaño de la ubicación).
* *GroupExpression* : una expresión sobre las columnas que proporciona un conjunto de valores distintivos. Habitualmente se trata de un nombre de columna que ya proporciona un conjunto restringido de valores. 
* *MakeSeriesParameters*: cero o más parámetros (separados por espacios) en forma de valor de *nombre* `=` *Value* que controlan el comportamiento. Se admiten los siguientes parámetros: 
  
  |Nombre           |Valores                                        |Descripción                                                                                        |
  |---------------|-------------------------------------|------------------------------------------------------------------------------|
  |`kind`          |`nonempty`                               |Genera el resultado predeterminado cuando la entrada del operador make-Series está vacía.|                                

**Devuelve**

Las filas de entrada están organizadas en grupos que tienen los mismos valores de las `by` expresiones y la expresión de inicio del `bin_at(` *AxisColumn* `, ` *paso*AxisColumn `, ` *start* `)` . A continuación, las funciones de agregación especificadas se calculan sobre cada grupo, generando una fila para cada grupo. El resultado contiene las `by` columnas *AxisColumn* y, además, al menos una columna para cada agregado calculado. (No se admite la agregación de varias columnas o resultados no numéricos).

Este resultado intermedio tiene tantas filas como combinaciones diferentes de valores de `by` Inicio de `bin_at(` *AxisColumn* `, ` *paso* `, ` *start* de AxisColumn y `)` .

Por último, las filas del resultado intermedio organizadas en grupos que tienen los mismos valores de las `by` expresiones y todos los valores agregados se organizan en matrices (valores de `dynamic` tipo). En cada agregación hay una columna que contiene la matriz con el mismo nombre. La última columna de la salida de la función de intervalo con todos los valores de *AxisColumn* . Su valor se repite para todas las filas. 

Tenga en cuenta que, debido al valor predeterminado de rellenar las bandejas que faltan, la tabla dinámica resultante tiene el mismo número de ubicaciones (es decir, valores agregados) para todas las series  

**Nota**

Aunque puede proporcionar expresiones arbitrarias para las expresiones de agregación y agrupación, resulta más eficaz usar nombres de columna simples.

**Sintaxis alternativa**

*T* `| make-series` [*columna* `=` ] *agregación* [ `default` `=` *DefaultValue*] [ `,` ...] `on` *AxisColumn* `in` `range(` *iniciar* `,` *detención* `,` *paso* `)` [ `by` [*columna* `=` ] *GroupExpression* [ `,` ...]]

La serie generada a partir de la sintaxis alternativa difiere de la sintaxis principal en 2 aspectos:
* El valor de *detención* es inclusivo.
* Discretización el eje de índice se genera con bin () y no bin_at (), lo que significa que no se garantiza que el *Inicio* se incluya en la serie generada.

Se recomienda usar la sintaxis principal de make-series y no la sintaxis alternativa.

**Distribución y orden aleatorio**

`make-series`admite `summarize` [sugerencias de shufflekey](shufflequery.md) mediante la sugerencia de sintaxis. shufflekey.

## <a name="list-of-aggregation-functions"></a>Lista de funciones de agregación

|Función|Descripción|
|--------|-----------|
|[any()](any-aggfunction.md)|Devuelve un valor no vacío aleatorio para el grupo.|
|[avg()](avg-aggfunction.md)|Valor promedio de devuelve en el grupo|
|[count ()](count-aggfunction.md)|Devuelve el recuento del grupo|
|[countif()](countif-aggfunction.md)|Devuelve el recuento con el predicado del grupo.|
|[dcount()](dcount-aggfunction.md)|Devuelve un recuento distinto aproximado de los elementos del grupo.|
|[Max ()](max-aggfunction.md)|Devuelve el valor máximo en el grupo.|
|[min ()](min-aggfunction.md)|Devuelve el valor mínimo en el grupo.|
|[stdev()](stdev-aggfunction.md)|Devuelve la desviación estándar en el grupo.|
|[sum()](sum-aggfunction.md)|Devuelve la suma de los elementos que tienen el grupo|
|[variance()](variance-aggfunction.md)|Devuelve la varianza en el grupo.|

## <a name="list-of-series-analysis-functions"></a>Lista de funciones de análisis de series

|Función|Descripción|
|--------|-----------|
|[series_fir()](series-firfunction.md)|Aplica un filtro de [respuesta finita al impulso](https://en.wikipedia.org/wiki/Finite_impulse_response)|
|[series_iir()](series-iirfunction.md)|Aplica un filtro de [respuesta infinita al impulso](https://en.wikipedia.org/wiki/Infinite_impulse_response)|
|[series_fit_line()](series-fit-linefunction.md)|Busca una línea recta que sea la mejor aproximación de la entrada|
|[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)|Busca una línea que sea la mejor aproximación de la entrada, devolviendo el objeto dinámico|
|[series_fit_2lines()](series-fit-2linesfunction.md)|Busca dos líneas que son la mejor aproximación de la entrada|
|[series_fit_2lines_dynamic()](series-fit-2lines-dynamicfunction.md)|Busca dos líneas que son la mejor aproximación de la entrada, devolviendo el objeto dinámico|
|[series_outliers()](series-outliersfunction.md)|Puntua los puntos de anomalía de una serie|
|[series_periods_detect()](series-periods-detectfunction.md)|Busca los períodos más significativos que existen en una serie temporal|
|[series_periods_validate()](series-periods-validatefunction.md)|Comprueba si una serie temporal contiene patrones periódicos de longitud determinada.|
|[series_stats_dynamic()](series-stats-dynamicfunction.md)|Devolver varias columnas con las estadísticas comunes (min/max/Variance/stdev/Average)|
|[series_stats()](series-statsfunction.md)|Genera un valor dinámico con las estadísticas comunes (min/max/Variance/stdev/Average)|
  
## <a name="list-of-series-interpolation-functions"></a>Lista de funciones de interpolación de series

|Función|Descripción|
|--------|-----------|
|[series_fill_backward()](series-fill-backwardfunction.md)|Realiza una interpolación de relleno hacia atrás de los valores que faltan en una serie|
|[series_fill_const()](series-fill-constfunction.md)|Reemplaza los valores que faltan en una serie con un valor constante especificado.|
|[series_fill_forward()](series-fill-forwardfunction.md)|Realiza la interpolación de relleno hacia delante de los valores que faltan en una serie|
|[series_fill_linear()](series-fill-linearfunction.md)|Realiza la interpolación lineal de los valores que faltan en una serie.|

* Nota: de forma predeterminada, las funciones de interpolación suponen `null` un valor que falta. Por lo tanto, se recomienda especificar `default=` *Double*( `null` ) en `make-series` si piensa usar funciones de interpolación para la serie. 

## <a name="example"></a>Ejemplo
  
 Una tabla que muestra las matrices de los números y los precios medios de cada fruta de cada uno de los proveedores ordenados por la marca de tiempo con el intervalo especificado. Hay una fila en la salida para cada combinación distinta de frutas y proveedores. Las columnas de salida muestran las frutas, proveedores y matrices de: Count, Average y toda la línea de tiempo (de 2016-01-01 a 2016-01-10). Todas las matrices se ordenan por la marca de tiempo correspondiente y todos los huecos se rellenan con los valores predeterminados (0 en este ejemplo). El resto de columnas de entrada se ignoran.
  
```kusto
T | make-series PriceAvg=avg(Price) default=0
on Purchase from datetime(2016-09-10) to datetime(2016-09-13) step 1d by Supplier, Fruit
```

:::image type="content" source="images/make-seriesoperator/makeseries.png" alt-text="Crea el":::  

 <!-- csl: https://help.kusto.windows.net:443/Samples --> 
```kusto
let data=datatable(timestamp:datetime, metric: real)
[
  datetime(2016-12-31T06:00), 50,
  datetime(2017-01-01), 4,
  datetime(2017-01-02), 3,
  datetime(2017-01-03), 4,
  datetime(2017-01-03T03:00), 6,
  datetime(2017-01-05), 8,
  datetime(2017-01-05T13:40), 13,
  datetime(2017-01-06), 4,
  datetime(2017-01-07), 3,
  datetime(2017-01-08), 8,
  datetime(2017-01-08T21:00), 8,
  datetime(2017-01-09), 2,
  datetime(2017-01-09T12:00), 11,
  datetime(2017-01-10T05:00), 5,
];
let interval = 1d;
let stime = datetime(2017-01-01);
let etime = datetime(2017-01-10);
data
| make-series avg(metric) on timestamp from stime to etime step interval 
```
  
|avg_metric|timestamp|
|---|---|
|[4,0, 3,0, 5,0, 0,0, 10,5, 4,0, 3,0, 8,0, 6,5]|["2017-01-01T00:00:00.0000000 Z", "2017-01-02T00:00:00.0000000 Z", "2017-01-03T00:00:00.0000000 Z", "2017-01-04T00:00:00.0000000 Z", "2017-01-05T00:00:00.0000000 Z", "2017-01-06T00:00:00.0000000 Z", "2017-01-07T00:00:00.0000000 Z", "2017-01-08T00:00:00.0000000 Z", "2017-01-09T00:00:00.0000000 Z"]|  


Cuando la entrada a `make-series` está vacía, el comportamiento predeterminado de `make-series` también genera un resultado vacío.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let data=datatable(timestamp:datetime, metric: real)
[
  datetime(2016-12-31T06:00), 50,
  datetime(2017-01-01), 4,
  datetime(2017-01-02), 3,
  datetime(2017-01-03), 4,
  datetime(2017-01-03T03:00), 6,
  datetime(2017-01-05), 8,
  datetime(2017-01-05T13:40), 13,
  datetime(2017-01-06), 4,
  datetime(2017-01-07), 3,
  datetime(2017-01-08), 8,
  datetime(2017-01-08T21:00), 8,
  datetime(2017-01-09), 2,
  datetime(2017-01-09T12:00), 11,
  datetime(2017-01-10T05:00), 5,
];
let interval = 1d;
let stime = datetime(2017-01-01);
let etime = datetime(2017-01-10);
data
| limit 0
| make-series avg(metric) default=1.0 on timestamp from stime to etime step interval 
| count 
```

|Count|
|---|
|0|


`kind=nonempty`El uso de en `make-series` producirá un resultado no vacío de los valores predeterminados:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let data=datatable(timestamp:datetime, metric: real)
[
  datetime(2016-12-31T06:00), 50,
  datetime(2017-01-01), 4,
  datetime(2017-01-02), 3,
  datetime(2017-01-03), 4,
  datetime(2017-01-03T03:00), 6,
  datetime(2017-01-05), 8,
  datetime(2017-01-05T13:40), 13,
  datetime(2017-01-06), 4,
  datetime(2017-01-07), 3,
  datetime(2017-01-08), 8,
  datetime(2017-01-08T21:00), 8,
  datetime(2017-01-09), 2,
  datetime(2017-01-09T12:00), 11,
  datetime(2017-01-10T05:00), 5,
];
let interval = 1d;
let stime = datetime(2017-01-01);
let etime = datetime(2017-01-10);
data
| limit 0
| make-series kind=nonempty avg(metric) default=1.0 on timestamp from stime to etime step interval 
```

|avg_metric|timestamp|
|---|---|
|[<br>  1,0,<br>  1,0,<br>  1,0,<br>  1,0,<br>  1,0,<br>  1,0,<br>  1,0,<br>  1,0,<br>  1.0<br>]|[<br>  "2017-01-01T00:00:00.0000000 Z",<br>  "2017-01-02T00:00:00.0000000 Z",<br>  "2017-01-03T00:00:00.0000000 Z",<br>  "2017-01-04T00:00:00.0000000 Z",<br>  "2017-01-05T00:00:00.0000000 Z",<br>  "2017-01-06T00:00:00.0000000 Z",<br>  "2017-01-07T00:00:00.0000000 Z",<br>  "2017-01-08T00:00:00.0000000 Z",<br>  "2017-01-09T00:00:00.0000000 Z"<br>]|

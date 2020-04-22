---
title: operador de la serie make- Microsoft Docs
description: En este artículo se describe el operador make-series en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/16/2020
ms.openlocfilehash: 3fa8f1693b56fc0820b9e0ba6b5f03a9363c9b98
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663719"
---
# <a name="make-series-operator"></a>Operador make-series

Cree una serie de valores agregados especificados a lo largo del eje especificado. 

```kusto
T | make-series sum(amount) default=0, avg(price) default=0 on timestamp from datetime(2016-01-01) to datetime(2016-01-10) step 1d by fruit, supplier
```

**Sintaxis**

*T* `| make-series` [*MakeSeriesParamters*] [`default` `=` *Columna* `=`]`,` *Agregación* [ Valor *predeterminado*] [ ...] `on` *AxisColumn* `from` *start*[ start`to` ] `step` [ *end*] *step* [`by` [*Column* `=`] *GroupExpression* [`,` ...]]

**Argumentos**

* *Column* : nombre opcional para una columna de resultados. El valor predeterminado es un nombre derivado de la expresión.
* *DefaultValue:* Valor predeterminado que se utilizará en lugar de los valores ausentes. Si no hay ninguna fila con valores específicos de *AxisColumn* y *GroupExpression,* en los resultados se asignará el elemento correspondiente de la matriz con un *valor predeterminado*. Si `default =` se omite *DefaultValue,* se asume 0. 
* *Agregación:* Una llamada a una `count()` función de [agregación](make-seriesoperator.md#list-of-aggregation-functions) como o `avg()`, con nombres de columna como argumentos. Consulte la [lista de funciones de agregación](make-seriesoperator.md#list-of-aggregation-functions). Tenga en cuenta que solo las funciones de agregación que devuelven resultados numéricos se pueden utilizar con `make-series` el operador.
* *AxisColumn:* Una columna en la que se ordenará la serie. Podría considerarse como línea de `datetime` tiempo, pero además se aceptan los tipos numéricos.
* *start*: (opcional) Se construirá el valor límite bajo de *AxisColumn* para cada serie. *start*, *end* y *step* se utilizan para crear una matriz de valores *AxisColumn* dentro de un intervalo determinado y utilizando el *paso*especificado. Todos los valores *de Agregación* se ordenan respectivamente a esta matriz. Esta matriz *AxisColumn* es también la última columna de salida de la salida con el mismo nombre que *AxisColumn*. Si no se especifica un valor *de inicio,* el inicio es la primera ubicación (paso) que tiene datos en cada serie.
* *end*: (opcional) El valor de límite alto (no inclusivo) de *AxisColumn*, el último índice de la serie temporal es menor que este valor (y se *iniciará* más múltiplo entero de *paso* que es menor que *end*). Si no se proporciona el valor *final,* será el límite superior de la última ubicación (paso) que tiene datos por cada serie.
* *paso*: La diferencia entre dos elementos consecutivos de la matriz *AxisColumn* (es decir, el tamaño de la ubicación).
* *GroupExpression* : una expresión sobre las columnas que proporciona un conjunto de valores distintivos. Habitualmente se trata de un nombre de columna que ya proporciona un conjunto restringido de valores. 
* *MakeSeriesParameters*: Cero o más parámetros (separados por espacios) en forma de *valor* de *nombre* `=` que controlan el comportamiento. Se admiten los siguientes parámetros: 
  
  |Nombre           |Valores                                        |Descripción                                                                                        |
  |---------------|-------------------------------------|------------------------------------------------------------------------------|
  |`kind`          |`nonempty`                               |Produce un resultado predeterminado cuando la entrada del operador make-series está vacía|                                

**Devuelve**

Las filas de entrada se organizan en `by` grupos `bin_at(`que tienen los mismos valores de las expresiones y la expresión*de inicio* `)` del*paso*`, ` *AxisColumn.*`, ` A continuación, las funciones de agregación especificadas se calculan sobre cada grupo, generando una fila para cada grupo. El resultado `by` contiene las columnas, la columna *AxisColumn* y también al menos una columna para cada agregado calculado. (Agregación que no se admiten varias columnas o resultados no numéricos.)

Este resultado intermedio tiene tantas filas como `by` combinaciones distintas de `bin_at(`y valores*de inicio* `)` de*paso*`, ` *AxisColumn.*`, `

Por último, las filas del resultado intermedio organizadas en grupos que tienen los mismos valores de las `by` expresiones y todos los valores agregados se organizan en matrices (valores de `dynamic` tipo). Para cada agregación hay una columna que contiene su matriz con el mismo nombre. La última columna de la salida de la función range con todos los valores *AxisColumn.* Su valor se repite para todas las filas. 

Tenga en cuenta que debido al relleno faltan ubicaciones por valor predeterminado, la tabla dinámica resultante tiene el mismo número de ubicaciones (es decir, valores agregados) para todas las series  

**Nota**

Aunque puede proporcionar expresiones arbitrarias para las expresiones de agregación y agrupación, es más eficaz usar nombres de columna simples.

**Sintaxis alternativa**

*T* `| make-series` [*Columna* `=``default` `=` ] *Agregación* [ *Valor predeterminado*] [`,` ...] `on` *Paso* `in` `range(`de`,` *detención* `)` `by` `=``,` *start* `,` de inicio de AxisColumn *[* [*Column* ] *GroupExpression* [ ...]]

La serie generada a partir de la sintaxis alternativa difiere de la sintaxis principal en 2 aspectos:
* El valor *stop* es inclusivo.
* Binning el eje de índice se genera con bin() y no bin_at(), lo que significa que no se garantiza que *start* se incluya en la serie generada.

Se recomienda utilizar la sintaxis principal de make-series y no la sintaxis alternativa.

**Distribución y aleatoria**

`make-series`admite `summarize` [sugerencias de shufflekey](shufflequery.md) mediante la sintaxis hint.shufflekey.

## <a name="list-of-aggregation-functions"></a>Lista de funciones de agregación

|Función|Descripción|
|--------|-----------|
|[any()](any-aggfunction.md)|Devuelve un valor aleatorio no vacío para el grupo|
|[avg()](avg-aggfunction.md)|Retuns valor medio en todo el grupo|
|[count()](count-aggfunction.md)|Recuento de devoluciones del grupo|
|[countif()](countif-aggfunction.md)|Devuelve count con el predicado del grupo|
|[dcount()](dcount-aggfunction.md)|Devuelve un recuento diferenciado aproximado de los elementos del grupo|
|[max()](max-aggfunction.md)|Devuelve el valor máximo en todo el grupo|
|[min()](min-aggfunction.md)|Devuelve el valor mínimo en todo el grupo|
|[stdev()](stdev-aggfunction.md)|Devuelve la desviación estándar en todo el grupo|
|[sum()](sum-aggfunction.md)|Devuelve la suma de los elementos con el grupo|
|[variance()](variance-aggfunction.md)|Devuelve la varianza en todo el grupo|

## <a name="list-of-series-analysis-functions"></a>Lista de funciones de análisis de series

|Función|Descripción|
|--------|-----------|
|[series_fir()](series-firfunction.md)|Aplica filtro [Finite Impulse Response](https://en.wikipedia.org/wiki/Finite_impulse_response)|
|[series_iir()](series-iirfunction.md)|Aplica el filtro [Infinite Impulse Response](https://en.wikipedia.org/wiki/Infinite_impulse_response)|
|[series_fit_line()](series-fit-linefunction.md)|Encuentra una línea recta que sea la mejor aproximación de la entrada|
|[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)|Busca una línea que sea la mejor aproximación de la entrada, devolviendo el objeto dinámico|
|[series_fit_2lines()](series-fit-2linesfunction.md)|Encuentra dos líneas que es la mejor aproximación de la entrada|
|[series_fit_2lines_dynamic()](series-fit-2lines-dynamicfunction.md)|Encuentra dos líneas que es la mejor aproximación de la entrada, devolviendo el objeto dinámico|
|[series_outliers()](series-outliersfunction.md)|Anota puntos de anomalía en una serie|
|[series_periods_detect()](series-periods-detectfunction.md)|Encuentra los períodos más significativos que existen en una serie temporal|
|[series_periods_validate()](series-periods-validatefunction.md)|Comprueba si una serie temporal contiene patrones periódicos de longitudes dadas|
|[series_stats_dynamic()](series-stats-dynamicfunction.md)|Devolver varias columnas con las estadísticas comunes (min/max/variance/stdev/average)|
|[series_stats()](series-statsfunction.md)|Genera un valor dinámico con las estadísticas comunes (min/max/variance/stdev/average)|
  
## <a name="list-of-series-interpolation-functions"></a>Lista de funciones de interpolación en serie
|Función|Descripción|
|--------|-----------|
|[series_fill_backward()](series-fill-backwardfunction.md)|Realiza la interpolación de relleno hacia atrás de los valores que faltan en una serie|
|[series_fill_const()](series-fill-constfunction.md)|Reemplaza los valores que faltan en una serie por un valor constante especificado|
|[series_fill_forward()](series-fill-forwardfunction.md)|Realiza la interpolación de relleno hacia delante de los valores que faltan en una serie|
|[series_fill_linear()](series-fill-linearfunction.md)|Realiza la interpolación lineal de los valores que faltan en una serie|

* Nota: Las funciones `null` de interpolación se suponen de forma predeterminada como un valor que falta. Por lo tanto, `default=`se`null`recomienda `make-series` especificar *double*( ) en si tiene la intención de utilizar funciones de interpolación para la serie. 

## <a name="example"></a>Ejemplo
  
 Una tabla que muestra matrices de los números y los precios medios de cada fruta de cada proveedor ordenados por la marca de tiempo con el rango especificado. Hay una fila en la salida para cada combinación distinta de fruta y proveedor. Las columnas de salida muestran la fruta, el proveedor y las matrices de: recuento, promedio y toda la línea de tiempo (desde 2016-01-01 hasta 2016-01-10). Todas las matrices se ordenan por la marca de tiempo respectiva y todas las brechas se rellenan con valores predeterminados (0 en este ejemplo). El resto de columnas de entrada se ignoran.
  
```kusto
T | make-series PriceAvg=avg(Price) default=0
on Purchase from datetime(2016-09-10) to datetime(2016-09-13) step 1d by Supplier, Fruit
```

:::image type="content" source="images/aggregations/makeseries.png" alt-text="Makeseries":::  
  
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
|[ 4.0, 3.0, 5.0, 0.0, 10.5, 4.0, 3.0, 8.0, 6.5 ]|[ "2017-01-01T00:00:00.0000000Z", "2017-01-02T00:00:00.0000000Z", "2017-01-03T00:00:00.0000000Z", "2017-01-04T00:00:0000000Z", "2017-01-05T00:00:00.0000000Z", "2017-01-06T00:00:00.00000000Z", "2017-01-07T00:00:00.0000000Z", "2017-01-07T00:00:00.0000000Z", "2017-01-07T00:00:00.0000000Z", "2017-01-07T00:00:00.00000000Z", "2017-01-07T00:00:00.00000000Z", "2 2017-01-08T00:00:00.0000000Z", "2017-01-09T00:00:00.0000000Z" ]|  


Cuando la `make-series` entrada está vacía, `make-series` el comportamiento predeterminado de produce un resultado vacío también.

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


El `kind=nonempty` `make-series` uso de in producirá un resultado no vacío de los valores predeterminados:

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
|[<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0,<br>  1.0<br>]|[<br>  "2017-01-01T00:00:00.0000000Z",<br>  "2017-01-02T00:00:00.0000000Z",<br>  "2017-01-03T00:00:00.0000000Z",<br>  "2017-01-04T00:00:00.0000000Z",<br>  "2017-01-05T00:00:00.0000000Z",<br>  "2017-01-06T00:00:00.0000000Z",<br>  "2017-01-07T00:00:00.0000000Z",<br>  "2017-01-08T00:00:00.0000000Z",<br>  "2017-01-09T00:00:00.0000000Z"<br>]|
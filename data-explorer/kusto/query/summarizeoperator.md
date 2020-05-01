---
title: 'resumir operador: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el operador de resumen en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/20/2020
ms.openlocfilehash: e81afc50c752ac1b673bcaac38a77c2712ce9ff4
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82619027"
---
# <a name="summarize-operator"></a>Operador summarize

Crea una tabla que agrega el contenido de la tabla de entrada.

```kusto
T | summarize count(), avg(price) by fruit, supplier
```

Una tabla que muestra el número y el precio medio de cada fruta de cada proveedor. Hay una fila en la salida para cada combinación distinta de frutas y proveedores. Las columnas de salida muestran el recuento, el precio medio, los frutos y el proveedor. El resto de columnas de entrada se ignoran.

```kusto
T | summarize count() by price_range=bin(price, 10.0)
```

Una tabla que muestra cuántos elementos tienen precios en cada intervalo [0,10.0], [10.0,20.0], y así sucesivamente. Este ejemplo tiene una columna para el recuento y otra para el intervalo de precios. El resto de columnas de entrada se ignoran.

**Sintaxis**

*T* `| summarize` [[*columna* `=`] *agregación* [`,` ...]] [`by` [*Columna* `=`] *GroupExpression* [`,` ...]]

**Argumentos**

* *Column* : nombre opcional para una columna de resultados. El valor predeterminado es un nombre derivado de la expresión.
* *Agregación:* Una llamada a una [función de agregación](summarizeoperator.md#list-of-aggregation-functions) `count()` como `avg()`o, con los nombres de columna como argumentos. Consulte la [lista de funciones de agregación](summarizeoperator.md#list-of-aggregation-functions).
* *GroupExpression* : una expresión sobre las columnas que proporciona un conjunto de valores distintivos. Normalmente, se trata de un nombre de columna que ya proporciona un conjunto limitado de valores, o bien `bin()` con una columna numérica o temporal como argumento. 

> [!NOTE]
> Cuando la tabla de entrada está vacía, el resultado depende de si se usa *GroupExpression* :
>
> * Si no se proporciona *GroupExpression* , la salida será una fila única (vacía).
> * Si se proporciona *GroupExpression* , la salida no tendrá ninguna fila.

**Devuelve**

Las filas de entrada están organizadas en grupos que tienen los mismos valores que las expresiones `by` . A continuación, las funciones de agregación especificadas se calculan sobre cada grupo, generando una fila para cada grupo. El resultado contiene las columnas `by` y también al menos una columna para cada agregación procesada. (Algunas funciones de agregación devuelven varias columnas.)

El resultado tiene tantas filas como combinaciones diferentes de `by` valores (que puede ser cero). Si no se proporciona ninguna clave de grupo, el resultado tiene un único registro.

Para resumir los intervalos de valores numéricos, `bin()` use para reducir los intervalos a valores discretos.

> [!NOTE]
> * Aunque puede proporcionar expresiones arbitrarias para las expresiones de agregación y las de agrupación, resulta más eficaz usar nombres de columna simples, o bien aplicar `bin()` a una columna numérica.
> * Ya no se admiten las bandejas horarias automáticas para las columnas DateTime. Use explícitamente discretización en su lugar. Por ejemplo, `summarize by bin(timestamp, 1h)`.

## <a name="list-of-aggregation-functions"></a>Lista de funciones de agregación

|Función|Descripción|
|--------|-----------|
|[any ()](any-aggfunction.md)|Devuelve un valor no vacío aleatorio para el grupo.|
|[anyif()](anyif-aggfunction.md)|Devuelve un valor no vacío aleatorio para el grupo (con predicado).|
|[arg_max()](arg-max-aggfunction.md)|Devuelve una o más expresiones cuando el argumento está maximizado.|
|[arg_min()](arg-min-aggfunction.md)|Devuelve una o más expresiones cuando el argumento está minimizado.|
|[AVG ()](avg-aggfunction.md)|Devuelve un valor promedio en todo el grupo|
|[avgif()](avgif-aggfunction.md)|Devuelve un valor promedio en el grupo (con predicado).|
|[binary_all_and](binary-all-and-aggfunction.md)|Devuelve el valor agregado mediante el binario `AND` del grupo|
|[binary_all_or](binary-all-or-aggfunction.md)|Devuelve el valor agregado mediante el binario `OR` del grupo|
|[binary_all_xor](binary-all-xor-aggfunction.md)|Devuelve el valor agregado mediante el binario `XOR` del grupo|
|[buildschema()](buildschema-aggfunction.md)|Devuelve el esquema mínimo que se Reda a todos `dynamic` los valores de la entrada.|
|[count ()](count-aggfunction.md)|Devuelve un recuento del grupo|
|[countif()](countif-aggfunction.md)|Devuelve un recuento con el predicado del grupo.|
|[dcount()](dcount-aggfunction.md)|Devuelve un recuento distinto aproximado de los elementos del grupo.|
|[dcountif()](dcountif-aggfunction.md)|Devuelve un recuento distinto aproximado de los elementos de grupo (con predicado).|
|[make_bag()](make-bag-aggfunction.md)|Devuelve un contenedor de propiedades de valores dinámicos dentro del grupo.|
|[make_bag_if()](make-bag-if-aggfunction.md)|Devuelve un contenedor de propiedades de valores dinámicos dentro del grupo (con el predicado)|
|[make_list()](makelist-aggfunction.md)|Devuelve una lista de todos los valores dentro del grupo.|
|[make_list_if()](makelistif-aggfunction.md)|Devuelve una lista de todos los valores dentro del grupo (con predicado).|
|[make_list_with_nulls()](make-list-with-nulls-aggfunction.md)|Devuelve una lista de todos los valores dentro del grupo, incluidos los valores NULL.|
|[make_set()](makeset-aggfunction.md)|Devuelve un conjunto de valores distintos dentro del grupo.|
|[make_set_if()](makesetif-aggfunction.md)|Devuelve un conjunto de valores distintos dentro del grupo (con predicado).|
|[Max ()](max-aggfunction.md)|Devuelve el valor máximo en el grupo.|
|[maxif()](maxif-aggfunction.md)|Devuelve el valor máximo en el grupo (con predicado).|
|[min ()](min-aggfunction.md)|Devuelve el valor mínimo en el grupo.|
|[minif()](minif-aggfunction.md)|Devuelve el valor mínimo en el grupo (con predicado).|
|[percentiles()](percentiles-aggfunction.md)|Devuelve el percentil aproximado del grupo.|
|[percentiles_array ()](percentiles-aggfunction.md)|Devuelve los percentiles aproximados del grupo|
|[percentilesw()](percentiles-aggfunction.md)|Devuelve el aproximado del percentil ponderado del grupo.|
|[percentilesw_array ()](percentiles-aggfunction.md)|Devuelve los percentiles ponderados aproximados del grupo|
|[stdev ()](stdev-aggfunction.md)|Devuelve la desviación estándar en el grupo.|
|[stdevif()](stdevif-aggfunction.md)|Devuelve la desviación estándar en el grupo (con predicado).|
|[SUM ()](sum-aggfunction.md)|Devuelve la suma de los elementos que tienen el grupo|
|[sumif()](sumif-aggfunction.md)|Devuelve la suma de los elementos que tienen el grupo (con el predicado)|
|[varianza ()](variance-aggfunction.md)|Devuelve la varianza en el grupo.|
|[varianceif()](varianceif-aggfunction.md)|Devuelve la varianza en el grupo (con predicado).|

## <a name="aggregates-default-values"></a>Agrega valores predeterminados

En la tabla siguiente se resumen los valores predeterminados de las agregaciones:

Operator       |Valor predeterminado                         
---------------|------------------------------------
 `count()`, `countif()`, `dcount()`, `dcountif()`         |   0                            
 `make_bag()`, `make_bag_if()`, `make_list()`, `make_list_if()`, `make_set()`, `make_set_if()` |    matriz dinámica vacía ([])          
 Todos los demás          |   null                           

 Cuando se usan estos agregados en entidades que incluyen valores NULL, los valores NULL se omitirán y no participarán en el cálculo (vea los ejemplos siguientes).

## <a name="examples"></a>Ejemplos

:::image type="content" source="images/summarizeoperator/summarize-price-by-supplier.png" alt-text="Resumir precio por frutas y proveedores":::

**Ejemplo**

Determine qué combinaciones únicas `ActivityType` de `CompletionStatus` y existen en una tabla. No hay ninguna función de agregación, solo las claves Group-by. La salida solo mostrará las columnas para los resultados:

```kusto
Activities | summarize by ActivityType, completionStatus
```

|`ActivityType`|`completionStatus`
|---|---
|`dancing`|`started`
|`singing`|`started`
|`dancing`|`abandoned`
|`singing`|`completed`

**Ejemplo**

Busca la marca de tiempo mínima y máxima de todos los registros de la tabla de actividades. No hay ninguna cláusula de agrupar por, así que hay una sola fila en la salida:

```kusto
Activities | summarize Min = min(Timestamp), Max = max(Timestamp)
```

|`Min`|`Max`
|---|---
|`1975-06-09 09:21:45` | `2015-12-24 23:45:00`

**Ejemplo**

Cree una fila para cada continente que muestre un recuento de las ciudades en las que se producen las actividades. Dado que hay pocos valores para "continente", no se necesita ninguna función de agrupación en la cláusula "by":

    Activities | summarize cities=dcount(city) by continent

|`cities`|`continent`
|---:|---
|`4290`|`Asia`|
|`3267`|`Europe`|
|`2673`|`North America`|


**Ejemplo**

En el ejemplo siguiente se calcula un histograma para cada tipo de actividad. Dado `Duration` que tiene muchos valores, `bin` use para agrupar sus valores en intervalos de 10 minutos:

```kusto
Activities | summarize count() by ActivityType, length=bin(Duration, 10m)
```

|`count_`|`ActivityType`|`length`
|---:|---|---
|`354`| `dancing` | `0:00:00.000`
|`23`|`singing` | `0:00:00.000`
|`2717`|`dancing`|`0:10:00.000`
|`341`|`singing`|`0:10:00.000`
|`725`|`dancing`|`0:20:00.000`
|`2876`|`singing`|`0:20:00.000`
|...

**Ejemplo para los valores predeterminados de agregados**

Cuando la entrada del `summarize` operador tiene al menos una clave Group-by vacía, su resultado también está vacío.

Cuando la entrada del `summarize` operador no tiene una clave Group-by vacía, el resultado son los valores predeterminados de los agregados utilizados `summarize`en:

```kusto
range x from 1 to 10 step 1
| where 1 == 2
| summarize any(x), arg_max(x, x), arg_min(x, x), avg(x), buildschema(todynamic(tostring(x))), max(x), min(x), percentile(x, 55), hll(x) ,stdev(x), sum(x), sumif(x, x > 0), tdigest(x), variance(x)
```

|any_x|max_x|max_x_x|min_x|min_x_x|avg_x|schema_x|max_x1|min_x1|percentile_x_55|hll_x|stdev_x|sum_x|sumif_x|tdigest_x|variance_x|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|||||||||||||||||

```kusto
range x from 1 to 10 step 1
| where 1 == 2
| summarize  count(x), countif(x > 0) , dcount(x), dcountif(x, x > 0)
```

|count_x|countif_|dcount_x|dcountif_x|
|---|---|---|---|
|0|0|0|0|

```kusto
range x from 1 to 10 step 1
| where 1 == 2
| summarize  make_set(x), make_list(x)
```

|set_x|list_x|
|---|---|
|[]|[]|

El promedio agregado suma todos los valores no NULL y solo cuenta los que participaron en el cálculo (no tendrán en cuenta los valores NULL).

```kusto
range x from 1 to 2 step 1
| extend y = iff(x == 1, real(null), real(5))
| summarize sum(y), avg(y)
```

|sum_y|avg_y|
|---|---|
|5|5|

El recuento normal contará los valores NULL: 

```kusto
range x from 1 to 2 step 1
| extend y = iff(x == 1, real(null), real(5))
| summarize count(y)
```

|count_y|
|---|
|2|

```kusto
range x from 1 to 2 step 1
| extend y = iff(x == 1, real(null), real(5))
| summarize make_set(y), make_set(y)
```

|set_y|set_y1|
|---|---|
|[5,0]|[5,0]|

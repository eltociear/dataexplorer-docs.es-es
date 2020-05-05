---
title: 'Consulta aleatoria: Azure Explorador de datos'
description: En este artículo se describe la consulta aleatoria en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bbae74ccdf0d9840a6ba4aa7427155f89c4af211
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82742014"
---
# <a name="shuffle-query"></a>Consulta aleatoria

La consulta aleatoria es una transformación de conservación de semántica para un conjunto de operadores que admiten la estrategia de orden aleatorio. En función de los datos reales, esta consulta puede producir un rendimiento considerablemente mejor.

Los operadores que admiten orden aleatorio en Kusto son [join](joinoperator.md), [resume](summarizeoperator.md)y [Make-series](make-seriesoperator.md).

Establezca una estrategia de consulta aleatoria mediante el `hint.strategy = shuffle` parámetro `hint.shufflekey = <key>`de consulta o.

Defina una [Directiva de particionamiento de datos](../management/partitioningpolicy.md) en la tabla. 

Establezca `shufflekey` como clave de partición hash de la tabla para mejorar el rendimiento, ya que se reduce la cantidad de datos necesarios para moverse entre los nodos del clúster.

**Sintaxis**

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.strategy = shuffle (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

```kusto
T
| summarize hint.strategy = shuffle count(), avg(price) by supplier
```

```kusto
T
| make-series hint.shufflekey = Fruit PriceAvg=avg(Price) default=0  on Purchase from datetime(2016-09-10) to datetime(2016-09-13) step 1d by Supplier, Fruit
```

Esta estrategia compartirá la carga en todos los nodos del clúster, donde cada nodo procesará una partición de los datos.
Resulta útil usar la estrategia de consulta aleatoria cuando la clave (`join` clave, `summarize` clave o `make-series` clave) tiene una cardinalidad alta y la estrategia de consulta normal alcanza los límites de la consulta.

**Diferencia entre Hint. Strategy = orden aleatorio y Hint. shufflekey = Key**

`hint.strategy=shuffle`significa que todas las claves ordenarán aleatoriamente el operador aleatorio.
Por ejemplo, en esta consulta:

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.strategy = shuffle (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId, ProcessId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

La función hash que ordena aleatoriamente los datos usará las dos claves ActivityId y ProcessId.

La consulta anterior es equivalente a:

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.shufflekey = ActivityId hint.shufflekey = ProcessId (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId, ProcessId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

Si la clave compuesta es demasiado única, pero cada clave no es lo suficientemente exclusiva, úsela `hint` para ordenar los datos por todas las claves del operador aleatorio.
Cuando el operador aleatorio tiene otros operadores que pueden usarse en orden `summarize` aleatorio `join`, como o, la consulta se vuelve más compleja y después Hint. Strategy = orden aleatorio no se aplicará.

Por ejemplo:

```kusto
T
| where Event=="Start"
| project ActivityId, Started=Timestamp, numeric_column
| summarize count(), numeric_column = any(numeric_column) by ActivityId
| join
    hint.strategy = shuffle (T
    | where Event=="End"
    | project ActivityId, Ended=Timestamp, numeric_column
)
on ActivityId, numeric_column
| extend Duration=Ended - Started
| summarize avg(Duration)
```

Si aplica el `hint.strategy=shuffle` (en lugar de omitir la estrategia durante el planeamiento de consultas) y ordena aleatoriamente los datos por la clave`ActivityId`compuesta `numeric_column`[,], el resultado no será correcto.
El `summarize` operador está en el lado izquierdo del `join` operador. Este operador agrupará por un subconjunto de `join` las claves, que en nuestro caso `ActivityId`es. Por lo tanto `summarize` , se agrupará por `ActivityId`la clave, mientras que los datos se particionan mediante la`ActivityId`clave `numeric_column`compuesta [,].
Orden aleatorio por la clave compuesta [`ActivityId`, `numeric_column`] no implica necesariamente que orden aleatorio para la clave `ActivityId` sea válido, y los resultados pueden ser incorrectos.

En este ejemplo se da por supuesto que la función hash usada para `binary_xor(hash(key1, 100) , hash(key2, 100))`una clave compuesta es:

```kusto

datatable(ActivityId:string, NumericColumn:long)
[
"activity1", 2,
"activity1" ,1,
]
| extend hash_by_key = binary_xor(hash(ActivityId, 100) , hash(NumericColumn, 100))
```

|ActivityId|NumericColumn|hash_by_key|
|---|---|---|
|activity1|2|56|
|activity1|1|65|

La clave compuesta para ambos registros se asignó a particiones diferentes (56 y 65), pero estos dos registros tienen el mismo valor de `ActivityId`. El `summarize` operador en el lado izquierdo de `join` espera valores similares de la columna `ActivityId` en la misma partición. Esta consulta generará resultados incorrectos.

Puede resolver este problema mediante el uso `hint.shufflekey` de para especificar la clave de orden aleatorio en `hint.shufflekey = ActivityId`la combinación con. Esta clave es común para todos los operadores que pueden realizarse en orden aleatorio.
En este caso, orden aleatorio es seguro, porque tanto `join` como `summarize` orden aleatorio por la misma clave. Por lo tanto, todos los valores similares estarán en la misma partición y los resultados serán correctos:

```kusto
T
| where Event=="Start"
| project ActivityId, Started=Timestamp, numeric_column
| summarize count(), numeric_column = any(numeric_column) by ActivityId
| join
    hint.shufflekey = ActivityId (T
    | where Event=="End"
    | project ActivityId, Ended=Timestamp, numeric_column
)
on ActivityId, numeric_column
| extend Duration=Ended - Started
| summarize avg(Duration)
```

|ActivityId|NumericColumn|hash_by_key|
|---|---|---|
|activity1|2|56|
|activity1|1|65|

En la consulta aleatoria, el número de particiones predeterminado es el número de nodos del clúster. Este número se puede invalidar mediante la sintaxis `hint.num_partitions = total_partitions`, que controlará el número de particiones.

Esta sugerencia es útil cuando el clúster tiene un número pequeño de nodos de clúster en los que el número de particiones predeterminadas es pequeño y la consulta sigue produciendo un error o tarda mucho tiempo de ejecución.

> [!Note]
> Tener muchas particiones puede consumir más recursos del clúster y degradar el rendimiento. En su lugar, elija cuidadosamente el número de partición empezando por la sugerencia. Strategy = orden aleatorio y empiece a aumentar gradualmente las particiones.

**Ejemplos**

En el ejemplo siguiente se muestra `summarize` cómo la orden aleatorio mejora considerablemente el rendimiento.

La tabla de origen tiene registros 150M y la cardinalidad de la clave Group by es de 10 millones, que está repartido por más de 10 nodos de clúster.

Al ejecutar la `summarize` estrategia normal, la consulta finaliza después de 1:08 y el pico de uso de memoria es de ~ 3 GB:

```kusto
orders
| summarize arg_max(o_orderdate, o_totalprice) by o_custkey 
| where o_totalprice < 1000
| count
```

|Count|
|---|
|1086|

Al usar la `summarize` estrategia de orden aleatorio, la consulta finaliza después de ~ 7 segundos y el pico de uso de memoria es de 0,43 GB:

```kusto
orders
| summarize hint.strategy = shuffle arg_max(o_orderdate, o_totalprice) by o_custkey 
| where o_totalprice < 1000
| count
```

|Count|
|---|
|1086|

En el ejemplo siguiente se muestra la mejora en un clúster que tiene dos nodos de clúster, la tabla tiene registros 60M y la cardinalidad de la clave Group by es 2 m.

La ejecución de la `hint.num_partitions` consulta sin usará solo dos particiones (como el número de nodos del clúster) y la siguiente consulta tardará entre 1:10 minutos:

```kusto
lineitem    
| summarize hint.strategy = shuffle dcount(l_comment), dcount(l_shipdate) by l_partkey 
| consume
```

Si se establece el número de particiones en 10, la consulta finalizará después de 23 segundos: 

```kusto
lineitem    
| summarize hint.strategy = shuffle hint.num_partitions = 10 dcount(l_comment), dcount(l_shipdate) by l_partkey 
| consume
```

En el ejemplo siguiente se muestra `join` cómo la orden aleatorio mejora considerablemente el rendimiento.

Los ejemplos se muestrearon en un clúster con 10 nodos en los que los datos se reparten en todos estos nodos.

La tabla izquierda tiene registros 15.000.000 donde la cardinalidad de la `join` clave es ~ 14m. El lado derecho de `join` es con los registros de 150M y la cardinalidad de `join` la clave es 10 m.
Al ejecutar la estrategia normal de `join`, la consulta finaliza después de aproximadamente 28 segundos y el pico de uso de memoria es de 1,43 GB:

```kusto
customer
| join
    orders
on $left.c_custkey == $right.o_custkey
| summarize sum(c_acctbal) by c_nationkey
```

Al usar la `join` estrategia de orden aleatorio, la consulta finaliza después de aproximadamente 4 segundos y el pico de uso de memoria es de 0,3 GB:

```kusto
customer
| join
    hint.strategy = shuffle orders
on $left.c_custkey == $right.o_custkey
| summarize sum(c_acctbal) by c_nationkey
```

Probar las mismas consultas en un conjunto de DataSet más grande donde el `join` lado izquierdo de es 150M y la cardinalidad de la clave es 148M. El lado derecho del `join` es 1,5 b y la cardinalidad de la clave es ~ 100m.

La consulta con la estrategia `join` predeterminada alcanza los límites de Kusto y el tiempo de espera después de 4 minutos.
Al usar la `join` estrategia de orden aleatorio, la consulta finaliza después de ~ 34 segundos y el pico de uso de memoria es de 1,23 GB.


En el ejemplo siguiente se muestra la mejora en un clúster que tiene dos nodos de clúster, la tabla tiene registros 60M y la cardinalidad `join` de la clave es 2 m.
La ejecución de la `hint.num_partitions` consulta sin usará solo dos particiones (como el número de nodos del clúster) y la siguiente consulta tardará entre 1:10 minutos:

```kusto
lineitem
| summarize dcount(l_comment), dcount(l_shipdate) by l_partkey
| join
    hint.shufflekey = l_partkey   part
on $left.l_partkey == $right.p_partkey
| consume
```

Si se establece el número de particiones en 10, la consulta finalizará después de 23 segundos: 

```kusto
lineitem
| summarize dcount(l_comment), dcount(l_shipdate) by l_partkey
| join
    hint.shufflekey = l_partkey  hint.num_partitions = 10    part
on $left.l_partkey == $right.p_partkey
| consume
```

---
title: 'Consulta aleatoria: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la consulta aleatoria en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 600e561937b779ff9dd10d5d82f5522d204466a0
ms.sourcegitcommit: 2e63c7c668c8a6200f99f18e39c3677fcba01453
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/24/2020
ms.locfileid: "82117699"
---
# <a name="shuffle-query"></a>Consulta aleatoria

La consulta aleatoria es una transformación de conservación de semántica para un conjunto de operadores que admite la estrategia de orden aleatorio que, dependiendo de los datos reales, puede producir un rendimiento considerablemente mejor.

Los operadores que admiten orden aleatorio en Kusto son [join](joinoperator.md), [resume](summarizeoperator.md) y [Make-series](make-seriesoperator.md).

La estrategia de consulta aleatoria se puede establecer mediante el `hint.strategy = shuffle` parámetro `hint.shufflekey = <key>`de consulta o.

También puede buscar en la definición de una [Directiva de particionamiento de datos](../management/partitioningpolicy.md) en la tabla. Las consultas en las `shufflekey` que también es la clave de partición hash de la tabla tienen un rendimiento mejor, ya que la cantidad de datos necesaria para moverse entre los nodos del clúster se reduce significativamente.

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
Resulta útil usar la estrategia de consulta aleatoria cuando la clave (clave`join` , `summarize` clave o `make-series` clave) tiene una cardinalidad alta, lo que provoca que la estrategia de consulta normal alcance los límites de la consulta.

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

Esta sugerencia se puede usar cuando está interesado en orden aleatorio los datos por todas las claves del operador aleatorio porque la clave compuesta es demasiado única pero cada clave no es lo suficientemente exclusiva.
Cuando el operador aleatorio tiene otros operadores shufflable como `summarize` o `join`, la consulta es más compleja y, a continuación, Hint. Strategy = aleatorio no se aplicará.

por ejemplo:

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

En este caso, si se aplica `hint.strategy=shuffle` (en lugar de omitir la estrategia durante el planeamiento de consultas) y se ordenan aleatoriamente los datos por la`ActivityId`clave `numeric_column`compuesta [,] el resultado no será correcto.
`summarize` Que está en el lado izquierdo del `join` groubs por un subconjunto de las `join` claves que es `ActivityId`. Significa que `summarize` se agrupará según la clave `ActivityId` , mientras que los datos se particionan mediante la clave compuesta`ActivityId`[ `numeric_column`,].
Orden aleatorio por la clave compuesta [`ActivityId`, `numeric_column`] no significa que se trata de un orden aleatorio válido para la clave ActivityId y los resultados pueden ser incorrectos.

Este ejemplo simplifica esto, suponiendo que la función hash usada para una clave compuesta es`binary_xor(hash(key1, 100) , hash(key2, 100))`

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



Como puede ver, la clave compuesta de ambos registros se ha asignado a distintas particiones 56 y 65, pero estos dos registros tienen el `ActivityId` mismo valor, lo `summarize` que significa que el en el `join` lado izquierdo de, que espera que los `ActivityId` valores similares de la columna estén en la misma partición, Defintely producir resultados incorrectos.

En este caso, `hint.shufflekey` soluciona este problema especificando la clave de orden aleatorio en la combinación, `hint.shufflekey = ActivityId` que es una clave común para todos los operadores de shuffelable.
En este caso, orden aleatorio es seguro, `join` y `summarize` se ordenan aleatoriamente por la misma clave para que todos los valores similares Defintely estén en la misma partición que los resultados son correctos:

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

En la consulta aleatoria, el número de particiones predeterminado es el número de nodos del clúster. Este número se puede invalidar mediante la sintaxis `hint.num_partitions = total_partitions` que controlará el número de particiones.

Esta sugerencia es útil cuando el clúster tiene un número pequeño de nodos de clúster en los que el número de particiones predeterminadas es pequeño y la consulta sigue produciendo un error o tarda mucho tiempo de ejecución.

Tenga en cuenta que la configuración de muchas particiones puede degradar el rendimiento y consumir más recursos del clúster, por lo que se recomienda elegir el número de partición con cuidado (empezando por la sugerencia. Strategy = orden aleatorio y comenzar a aumentar las particiones gradualmente).

**Ejemplos**

En el ejemplo siguiente se muestra `summarize` cómo la orden aleatorio mejora considerablemente el rendimiento.

La tabla de origen tiene registros de 150M y la cardinalidad de la clave agrupar por es 10 millones, que se distribuye en 10 nodos de clúster.

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

Al usar la `summarize` estrategia de orden aleatorio, la consulta finaliza después de ~ 7 segundos y el pico de uso de memoria es de 0.43 GB:

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

La ejecución de la `hint.num_partitions` consulta sin usará solo 2 particiones (como el número de nodos del clúster) y la siguiente consulta tardará entre 1:10 minutos:

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

La tabla izquierda tiene registros 15.000.000 donde la cardinalidad de la `join` clave es ~ 14m, el lado derecho del `join` es con registros 150M y la cardinalidad de la `join` clave es 10 millones.
Al ejecutar la estrategia normal de `join`, la consulta finaliza después de aproximadamente 28 segundos y el pico de uso de memoria es de 1.43 GB:

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

Probar las mismas consultas en un conjunto de DataSet mayor en el que `join` el lado izquierdo del es 150M y la cardinalidad de la clave es 148M, `join` el lado derecho del es 1,5 b y la cardinalidad de la clave es ~ 100m.

La consulta con la estrategia `join` predeterminada alcanza los límites de kusto y el tiempo de espera después de 4 minutos.
Al usar la `join` estrategia de orden aleatorio, la consulta finaliza después de ~ 34 segundos y el pico de uso de memoria es de 1,23 GB.


En el ejemplo siguiente se muestra la mejora en un clúster que tiene dos nodos de clúster, la tabla tiene registros 60M y la cardinalidad de la `join` clave es 2 m.
La ejecución de la `hint.num_partitions` consulta sin usará solo 2 particiones (como el número de nodos del clúster) y la siguiente consulta tardará entre 1:10 minutos:

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

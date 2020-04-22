---
title: 'Consulta aleatoria: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la consulta aleatoria en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c687d495a41a5f73ac8dbca15d93729f2132a556
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81662970"
---
# <a name="shuffle-query"></a>Consulta aleatoria

La consulta aleatoria es una transformación de conservación semántica para un conjunto de operadores que admite la estrategia aleatoria que, dependiendo de los datos reales, puede producir un rendimiento considerablemente mejor.

Los operadores que admiten el barajado en Kusto se [unen,](joinoperator.md) [resumen](summarizeoperator.md) y [make-series](make-seriesoperator.md).

La estrategia de consulta aleatoria `hint.strategy = shuffle` se `hint.shufflekey = <key>`puede establecer mediante el parámetro de consulta o .

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

Esta estrategia compartirá la carga en todos los nodos del clúster donde cada nodo procesará una partición de los datos.
Es útil utilizar la estrategia de consulta`join` aleatoria `summarize` cuando `make-series` la clave (clave, clave o clave) tiene una alta cardinalidad que hace que la estrategia de consulta regular llegue a los límites de consulta.

**Diferencia entre hint.strategy-shuffle y hint.shufflekey - clave**

`hint.strategy=shuffle`significa que el operador aleatorio será barajado por todas las teclas.
Por ejemplo, en esta consulta:

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.strategy = shuffle (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId, ProcessId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

La función hash que baraja los datos usará las claves ActivityId y ProcessId.

La consulta anterior es equivalente a:

```kusto
T | where Event=="Start" | project ActivityId, Started=Timestamp
| join hint.shufflekey = ActivityId hint.shufflekey = ProcessId (T | where Event=="End" | project ActivityId, Ended=Timestamp)
  on ActivityId, ProcessId
| extend Duration=Ended - Started
| summarize avg(Duration)
```

Esta sugerencia se puede utilizar cuando está interesado en barajar los datos por todas las claves del operador aleatorio porque la clave compuesta es demasiado única, pero cada clave no es lo suficientemente única.
Cuando el operador aleatorio tiene otros operadores shufflable como `summarize` o `join`, la consulta se vuelve más compleja y, a continuación, hint.strategy-shuffle no se aplicará.

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

En este caso, si `hint.strategy=shuffle` aplicamos el (en lugar de ignorar la estrategia durante la`ActivityId` `numeric_column`planificación de consultas) y barajar los datos por la clave compuesta [ , ] el resultado no será correcto.
El `summarize` que está en el `join` lado izquierdo de los `join` groubs por un subconjunto de las claves que es `ActivityId`. Esto significa `summarize` que el grupo `ActivityId` de voluntad por la clave`ActivityId`mientras los datos se particionan por la clave compuesta [ , `numeric_column`].
Desviar por la tecla`ActivityId`compuesta `numeric_column`[ , ] no significa que sea un barajado válido para la clave ActivityId y los resultados pueden ser incorrectos.

Este ejemplo simplifica esta suponiendo que la función hash utilizada para una clave compuesta es`binary_xor(hash(key1, 100) , hash(key2, 100))`

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



Como puede ver, la clave compuesta para ambos registros se asignó a diferentes `ActivityId` particiones `summarize` 56 y 65, pero estos dos registros tienen el mismo valor, lo que significa que el en el lado izquierdo de la `join` que espera valores similares de la columna `ActivityId` en la misma partición significará resultados incorrectos.

En este `hint.shufflekey` caso, resuelve este problema especificando la `hint.shufflekey = ActivityId` clave aleatoria en la combinación a la que es una clave común para todos los operadores shuffelable.
En este caso, el barajado `join` es `summarize` seguro, tanto como baraja por la misma clave por lo que todos los valores similares definirán defintely en la misma partición los resultados son correctos:

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

En la consulta aleatoria, el número de particiones predeterminado es el número de nodos del clúster. Este número se puede invalidar `hint.num_partitions = total_partitions` mediante la sintaxis que controlará el número de particiones.

Esta sugerencia es útil cuando el clúster tiene un pequeño número de nodos de clúster donde el número de particiones predeterminado también será pequeño y la consulta sigue fallando o tarda mucho tiempo en la ejecución.

Tenga en cuenta que la configuración de muchas particiones puede degradar el rendimiento y consumir más recursos de clúster, por lo que se recomienda elegir el número de partición con cuidado (comenzando con hint.strategy - shuffle y empezar a aumentar las particiones gradualmente).

**Ejemplos**

En el ejemplo `summarize` siguiente se muestra cómo shuffle mejora considerablemente el rendimiento.

La tabla de origen tiene registros 150M y la cardinalidad del grupo por clave es 10M que se distribuye en 10 nodos de clúster.

Ejecutando `summarize` la estrategia regular, la consulta finaliza después de 1:08 y el pico de uso de memoria es de 3 GB:

```kusto
orders
| summarize arg_max(o_orderdate, o_totalprice) by o_custkey 
| where o_totalprice < 1000
| count
```

|Count|
|---|
|1086|

Durante el `summarize` uso de la estrategia aleatoria, la consulta finaliza después de 7 segundos y el pico de uso de memoria es de 0,43 GB:

```kusto
orders
| summarize hint.strategy = shuffle arg_max(o_orderdate, o_totalprice) by o_custkey 
| where o_totalprice < 1000
| count
```

|Count|
|---|
|1086|

En el ejemplo siguiente se muestra la mejora en un clúster que tiene 2 nodos de clúster, la tabla tiene 60M registros y la cardinalidad del grupo por clave es 2M.

La ejecución `hint.num_partitions` de la consulta sin utilizará solo 2 particiones (como número de nodos de clúster) y la siguiente consulta tardará 1:10 minutos:

```kusto
lineitem    
| summarize hint.strategy = shuffle dcount(l_comment), dcount(l_shipdate) by l_partkey 
| consume
```

estableciendo el número de particiones en 10, la consulta finalizará después de 23 segundos: 

```kusto
lineitem    
| summarize hint.strategy = shuffle hint.num_partitions = 10 dcount(l_comment), dcount(l_shipdate) by l_partkey 
| consume
```

En el ejemplo `join` siguiente se muestra cómo shuffle mejora considerablemente el rendimiento.

Los ejemplos se muestrearon en un clúster con 10 nodos donde los datos se distribuyen entre todos estos nodos.

La tabla izquierda tiene registros de 15M donde la cardinalidad de `join` la `join` clave es de 14m, el `join` lado derecho de la es con 150M registros y la cardinalidad de la clave es 10M.
Ejecutando la estrategia `join`regular de la , la consulta termina después de 28 segundos y el pico de uso de memoria es de 1,43 GB:

```kusto
customer
| join
    orders
on $left.c_custkey == $right.o_custkey
| summarize sum(c_acctbal) by c_nationkey
```

Durante el `join` uso de la estrategia aleatoria, la consulta finaliza después de 4 segundos y el pico de uso de memoria es de 0,3 GB:

```kusto
customer
| join
    hint.strategy = shuffle orders
on $left.c_custkey == $right.o_custkey
| summarize sum(c_acctbal) by c_nationkey
```

Probar las mismas consultas en un conjunto `join` de datos más grande donde el lado izquierdo de la es `join` 150M y la cardinalidad de la clave es 148M, el lado derecho de la es 1.5B y la cardinalidad de la clave es 100M.

La consulta con `join` la estrategia predeterminada alcanza los límites de kusto y el tiempo de espera después de 4 minutos.
Durante el `join` uso de la estrategia de reproducción aleatoria, la consulta finaliza después de 34 segundos y el pico de uso de memoria es de 1,23 GB.


En el ejemplo siguiente se muestra la mejora en un clúster que tiene 2 nodos `join` de clúster, la tabla tiene 60M registros y la cardinalidad de la clave es 2M.
La ejecución `hint.num_partitions` de la consulta sin utilizará solo 2 particiones (como número de nodos de clúster) y la siguiente consulta tardará 1:10 minutos:

```kusto
lineitem
| summarize dcount(l_comment), dcount(l_shipdate) by l_partkey
| join
    hint.shufflekey = l_partkey   part
on $left.l_partkey == $right.p_partkey
| consume
```

estableciendo el número de particiones en 10, la consulta finalizará después de 23 segundos: 

```kusto
lineitem
| summarize dcount(l_comment), dcount(l_shipdate) by l_partkey
| join
    hint.shufflekey = l_partkey  hint.num_partitions = 10    part
on $left.l_partkey == $right.p_partkey
| consume
```
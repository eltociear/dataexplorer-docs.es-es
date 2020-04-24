---
title: 'operador de partición: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el operador de partición en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0a9ef31f68a989fffe42d9b54800e9305e51f4ed
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511380"
---
# <a name="partition-operator"></a>Operador partition

El operador de partición divide su tabla de entrada en varias subtablas según los valores de la columna especificada, ejecuta una subconsulta sobre cada subtabla y genera una única tabla de salida que es la unión de los resultados de todas las subconsultas. 

```kusto
T | partition by Col1 ( top 10 by MaxValue )

T | partition by Col1 { U | where Col2=toscalar(Col1) }
```

**Sintaxis**

*T* `|` T `partition` [*PartitionParameters*] `by` *Columna* `(` *ContextualSubconsulta*`)`

*T* `|` T `partition` [*PartitionParameters*] `by` *Subconsulta* de *columna* `{``}`

**Argumentos**

* *T*: El origen tabular cuyos datos deben ser procesados por el operador.

* *Columna*: el nombre de una columna en *T* cuyos valores determinan cómo se va a particionar la tabla de entrada. Consulte **las notas** a continuación.

* *ContextualSubquery*: Una expresión tabular, cuyo `partition` origen es el origen del operador, con el ámbito de un único valor de *clave.*

* *Subconsulta*: Una expresión tabular sin origen. El valor *de clave* `toscalar()` se puede obtener mediante llamada.

* *PartitionParameters*: Cero o más parámetros (separados por espacios) en forma de: *Valor* de *nombre* `=` que controlan el comportamiento del operador. Se admiten los siguientes parámetros:

  |NOMBRE               |Valores         |Descripción|
  |-------------------|---------------|-----------|
  |`hint.materialized`|`true`,`false` |Si se `true` establece para materializar `partition` la fuente `false`del operador (predeterminado: )|
  |`hint.concurrency`|*Número*|Sugerencias al sistema cuántas subconsultas simultáneas del `partition` operador se deben ejecutar en paralelo. *Predeterminado:* Cantidad de núcleos de CPU en el nodo único del clúster (2 a 16).|
  |`hint.spread`|*Número*|Sugerencias al sistema cuántos nodos debe`partition` utilizar la ejecución simultánea de subconsultas. *Predeterminado*: 1.|

**Devuelve**

El operador devuelve una unión de los resultados de aplicar la subconsulta a cada partición de los datos de entrada.

**Notas**

* El operador de partición está actualmente limitado por el número de particiones.
  Se pueden crear hasta 64 particiones distintas.
  El operador producirá un error si la columna de partición (*Columna*) tiene más de 64 valores distintos.

* La subconsulta hace referencia a la partición de entrada implícitamente (no hay ningún "nombre" para la partición en la subconsulta). Para hacer referencia a la partición de entrada varias veces dentro de la subconsulta, utilice el [operador as](asoperator.md), como en **Ejemplo: referencia de partición** a continuación.

**Ejemplo: caso anidado superior**

En algunos casos - es más performante y `partition` más fácil escribir la consulta usando `summarize` operator `top` en lugar de `W`usar [ `top-nested` operator](topnestedoperator.md) El siguiente ejemplo ejecuta una subconsulta de cálculo y para cada uno de los estados a partir de: (WYOMING, WASHINGTON, WEST VIRGINIA, WISCONSIN)

```kusto
StormEvents
| where State startswith 'W'
| partition by State 
(
    summarize Events=count(), Injuries=sum(InjuriesDirect) by EventType, State
    | top 3 by Events 
) 

```
|EventType|State|Events|Lesiones|
|---|---|---|---|
|Granizo|Wyoming|108|0|
|Viento alto|Wyoming|81|5|
|Tormenta de invierno|Wyoming|72|0|
|Nieve pesada|Washington|82|0|
|Viento alto|Washington|58|13|
|Wildfire|Washington|29|0|
|Viento de tormenta|VIRGINIA OCCIDENTAL|180|1|
|Granizo|VIRGINIA OCCIDENTAL|103|0|
|Clima de invierno|VIRGINIA OCCIDENTAL|88|0|
|Viento de tormenta|Wisconsin|416|1|
|Tormenta de invierno|Wisconsin|310|0|
|Granizo|Wisconsin|303|1|

**Ejemplo: consultar particiones de datos no superpuestas**

A veces resulta útil (perf-wise) ejecutar una subconsulta compleja sobre particiones de datos no superpuestas en un estilo de mapa/reducción. En el ejemplo siguiente se muestra cómo crear una distribución manual de agregación en 10 particiones.

```kusto
StormEvents
| extend p = hash(EventId, 10)
| partition by p
(
    summarize Count=count() by Source 
)
| summarize Count=sum(Count) by Source
| top 5 by Count
```

|Source|Count|
|---|---|
|Observador entrenado|12770|
|Cuerpos de seguridad|8570|
|Público|6157|
|Administrador de emergencia|4900|
|COOP Observer|3039|

**Ejemplo: particionamiento en tiempo de consulta**

En el ejemplo siguiente se muestra cómo se puede particionar la consulta en particiones N-10, donde cada partición calcula su propio Count y todo más adelante se resume en TotalCount.

```kusto
let N = 10;                 // Number of query-partitions
range p from 0 to N-1 step 1  // 
| partition by p            // Run the sub-query partitioned 
{
    StormEvents 
    | where hash(EventId, N) == toscalar(p) // Use toscalar() to fetch partition key value
    | summarize Count = count()
}
| summarize TotalCount=sum(Count) 
```

|TotalCount|
|---|
|59066|


**Ejemplo: partición-referencia**

En el ejemplo siguiente se muestra cómo se puede utilizar el [operador as](asoperator.md) para dar un "nombre" a cada partición de datos y, a continuación, reutilizar ese nombre dentro de la subconsulta:

```kusto
T
| partition by Dim
(
    as Partition
    | extend MetricPct = Metric * 100.0 / toscalar(Partition | summarize sum(Metric))
)
```

**Ejemplo: subconsulta compleja oculta por una llamada de función**

La misma técnica se puede aplicar con subconsultas mucho más complejas. Para simplificar la sintaxis, se puede ajustar la subconsulta en una llamada de función:

```kusto
let partition_function = (T:(Source:string)) 
{
    T
    | summarize Count=count() by Source
};
StormEvents
| extend p = hash(EventId, 10)
| partition by p
(
    invoke partition_function()
)
| summarize Count=sum(Count) by Source
| top 5 by Count
```

|Source|Count|
|---|---|
|Observador entrenado|12770|
|Cuerpos de seguridad|8570|
|Público|6157|
|Administrador de emergencia|4900|
|COOP Observer|3039|
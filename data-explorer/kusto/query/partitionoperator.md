---
title: 'operador de partición: Azure Explorador de datos'
description: En este artículo se describe el operador de partición en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 417d4afb74e9170301baebde6be73d97df097f0f
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271543"
---
# <a name="partition-operator"></a>Operador partition

El operador Partition divide la tabla de entrada en varias subtablas según los valores de la columna especificada, ejecuta una subconsulta en cada subtabla y genera una única tabla de salida que es la Unión de los resultados de todas las subconsultas. 

```kusto
T | partition by Col1 ( top 10 by MaxValue )

T | partition by Col1 { U | where Col2=toscalar(Col1) }
```

**Sintaxis**

*T* `|` `partition` [*PartitionParameters*] `by` *columna* `(` *ContextualSubquery*`)`

*T* `|` `partition` [*PartitionParameters*] `by` *Column* `{` *subconsulta* de columna`}`

**Argumentos**

* *T*: el origen tabular cuyos datos se van a procesar por el operador.

* *Columna*: el nombre de una columna en *T* cuyos valores determinan cómo se va a particionar la tabla de entrada. Vea las **notas** siguientes.

* *ContextualSubquery*: expresión tabular, que es el origen del `partition` operador, con ámbito para un valor de *clave* única.

* *Subconsulta*: expresión tabular sin origen. El valor de *clave* se puede obtener mediante una `toscalar()` llamada a.

* *PartitionParameters*: cero o más parámetros (separados por espacios) con el formato: valor de *nombre* `=` *Value* que controla el comportamiento del operador. Se admiten los siguientes parámetros:

  |Nombre               |Valores         |Descripción|
  |-------------------|---------------|-----------|
  |`hint.materialized`|`true`,`false` |Si se establece en `true` , materializará el origen del `partition` operador (valor predeterminado: `false` )|
  |`hint.concurrency`|*Números*|Sugiere al sistema cuántas subconsultas simultáneas del `partition` operador se deben ejecutar en paralelo. *Valor predeterminado*: cantidad de núcleos de CPU en el único nodo del clúster (de 2 a 16).|
  |`hint.spread`|*Números*|Sugiere al sistema el número de nodos que debe usar la ejecución simultánea de `partition` subconsultas. *Valor predeterminado*: 1.|

**Devuelve**

El operador devuelve una Unión de los resultados de aplicar la subconsulta a cada partición de los datos de entrada.

**Notas**

* El operador Partition está limitado actualmente por el número de particiones.
  Se pueden crear hasta 64 particiones distintas.
  El operador producirá un error si la columna de partición (*columna*) tiene más de 64 valores distintos.

* La subconsulta hace referencia a la partición de entrada implícitamente (no hay "Name" para la partición en la subconsulta). Para hacer referencia a la partición de entrada varias veces dentro de la subconsulta, utilice el [operador as](asoperator.md), como en el **ejemplo siguiente: Partition-Reference** .

**Ejemplo: Top-Nested Case**

En algunos casos, es más eficaz y más fácil escribir consultas con `partition` el operador, en lugar de usar el [ `top-nested` operador](topnestedoperator.md) . en el ejemplo siguiente se ejecuta una subconsulta que calcula `summarize` y `top` para cada uno de los Estados a partir de `W` : (Wyoming, Washington, West Virginia, Wisconsin)

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| where State startswith 'W'
| partition by State 
(
    summarize Events=count(), Injuries=sum(InjuriesDirect) by EventType, State
    | top 3 by Events 
) 

```
|EventType|Estado|Events|Autolesiones|
|---|---|---|---|
|Granizo|WYOMING|108|0|
|Viento alta|WYOMING|81|5|
|Storm Storm|WYOMING|72|0|
|Nieve pesado|WASHINGTON|82|0|
|Viento alta|WASHINGTON|58|13|
|Descontrolados|WASHINGTON|29|0|
|Viento de tormenta|VIRGINIA OCCIDENTAL|180|1|
|Granizo|VIRGINIA OCCIDENTAL|103|0|
|Tiempo de invierno|VIRGINIA OCCIDENTAL|88|0|
|Viento de tormenta|WISCONSIN|416|1|
|Storm Storm|WISCONSIN|310|0|
|Granizo|WISCONSIN|303|1|

**Ejemplo: consulta de particiones de datos no superpuestas**

A veces, resulta útil (rendimiento) ejecutar una subconsulta compleja en particiones de datos no superpuestas en un estilo de asignación/reducción. En el ejemplo siguiente se muestra cómo crear una distribución manual de agregación en 10 particiones.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
|Observador de COOP|3039|

**Ejemplo: creación de particiones en tiempo de consulta**

En el ejemplo siguiente se muestra cómo se puede crear una partición de la consulta en N = 10 particiones, donde cada partición calcula su propio recuento y, posteriormente, se resume en TotalCount.

<!-- csl: https://help.kusto.windows.net/Samples -->
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


**Ejemplo: Partition-Reference**

En el ejemplo siguiente se muestra cómo se puede usar el [operador as](asoperator.md) para asignar un "nombre" a cada partición de datos y, a continuación, volver a usar ese nombre dentro de la subconsulta:

```kusto
T
| partition by Dim
(
    as Partition
    | extend MetricPct = Metric * 100.0 / toscalar(Partition | summarize sum(Metric))
)
```

**Ejemplo: subconsulta compleja oculta por una llamada de función**

Se puede aplicar la misma técnica con subconsultas mucho más complejas. Para simplificar la sintaxis, se puede incluir la subconsulta en una llamada de función:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
|Observador de COOP|3039|
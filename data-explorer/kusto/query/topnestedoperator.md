---
title: 'operador superior anidado: Azure Explorador de datos'
description: En este artículo se describe el operador de nivel superior anidado en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3fc4cfa307a283c4eb21ba60e3b83ba89b574757
ms.sourcegitcommit: aaada224e2f8824b51e167ddb6ff0bab92e5485f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2020
ms.locfileid: "84626690"
---
# <a name="top-nested-operator"></a>top-nested operator

Genera una selección jerárquica de agregación y valores superiores, donde cada nivel es un perfeccionamiento de la anterior.

```kusto
T | top-nested 3 of Location with others="Others" by sum(MachinesNumber), top-nested 4 of bin(Timestamp,5m) by sum(MachinesNumber)
```

El `top-nested` operador acepta datos tabulares como entrada y una o varias cláusulas de agregación.
La primera cláusula de agregación (más a la izquierda) divide los registros de entrada en particiones, de acuerdo con los valores únicos de alguna expresión sobre esos registros. A continuación, la cláusula mantiene un número determinado de registros que maximizan o minimizan esta expresión sobre los registros. A continuación, la siguiente cláusula de agregación aplica una función similar, de manera anidada. Cada cláusula siguiente se aplica a la partición generada por la cláusula anterior. Este proceso continúa para todas las cláusulas de agregación.

Por ejemplo, el `top-nested` operador se puede usar para responder a la siguiente pregunta: "para una tabla que contenga cifras de ventas, como país, vendedor y importe vendido: ¿Cuáles son los cinco países principales por ventas? ¿Cuáles son los tres principales vendedores en cada uno de estos países? "

**Sintaxis**

*T* `|` `top-nested` *TopNestedClause2* [ `,` *TopNestedClause2*...]

Donde *TopNestedClause* tiene la siguiente sintaxis:

[*N*] `of` [ *`ExprName`* `=` ] *`Expr`* [ `with` `others` `=` *`ConstExpr`* ] `by` [] *`AggName`* `=` *`Aggregation`* [ `asc`  |  `desc` ]

**Argumentos**

Para cada *TopNestedClause*:

* *`N`*: Un literal de tipo `long` que indica el número de valores superiores que se devolverán para este nivel de jerarquía.
  Si se omite, se devolverán todos los valores distintos.

* *`ExprName`*: Si se especifica, establece el nombre de la columna de salida correspondiente a los valores de *`Expr`* .

* *`Expr`*: Una expresión sobre el registro de entrada que indica el valor que se va a devolver para este nivel de jerarquía.
  Normalmente es una referencia de columna para la entrada tabular (*T*) o algún cálculo (como `bin()` ) sobre dicha columna.

* *`ConstExpr`*: Si se especifica, para cada nivel de jerarquía, se agregará 1 registro con el valor que es la agregación de todos los registros que no "lo convierten en la parte superior".

* *`AggName`*: Si se especifica, este identificador establece el nombre de columna en la salida para el valor de *agregación*.

* *`Aggregation`*: Una expresión numérica que indica la agregación que se va a aplicar a todos los registros que comparten el mismo valor de *`Expr`* . El valor de esta agregación determina cuál de los registros resultantes es "Top".
  
  Se admiten las siguientes funciones de agregación:
   * [SUM ()](sum-aggfunction.md),
   * [Count ()](count-aggfunction.md),
   * [Max ()](max-aggfunction.md),
   * [min ()](min-aggfunction.md),
   * [DCont ()](dcountif-aggfunction.md),
   * [AVG ()](avg-aggfunction.md),
   * [percentil ()](percentiles-aggfunction.md)y
   * [percentilew ()](percentiles-aggfunction.md). También se admite cualquier combinación algebraica de las agregaciones.

* `asc`o `desc` (valor predeterminado) pueden parecer controlar si la selección es realmente de la "parte inferior" o la "parte superior" del intervalo de valores agregados.

**Devuelve**

Este operador devuelve una tabla que tiene dos columnas para cada cláusula de agregación:

* Una columna contiene los valores distintos del cálculo de la cláusula *`Expr`* (con el nombre de columna *ExprName* si se especifica)

* Una columna contiene el resultado del cálculo de *agregación* (con el nombre de columna *AggregationName* si se especifica)

**Comentarios**

Las columnas de entrada que no se especifican como valores no se emiten *`Expr`* .
Para obtener todos los valores de un determinado nivel, agregue un recuento de agregaciones que:

* Omite el valor de *N*
* Utiliza el nombre de columna como el valor de*`Expr`*
* Utiliza `Ignore=max(1)` como agregación y, después, omite (o saca el proyecto) la columna `Ignore` .

El número de registros puede aumentar exponencialmente con el número de cláusulas de agregación ((N1 + 1) \* (N2 + 1) \* ...). El crecimiento de los registros es incluso más rápido si no se especifica ningún límite *N* . Tenga en cuenta que este operador puede consumir una cantidad considerable de recursos.

Si la distribución de la agregación es considerablemente no uniforme, limite el número de valores distintos que se van a devolver (mediante *N*) y use la `with others=` opción *ConstExpr* para obtener una indicación del "peso" de todos los demás casos.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation by sum(BeginLat)
```

|State|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|KANSAS|87771.2355000001|Cuerpos de seguridad|18744,823|FT SCOTT|264,858|
|KANSAS|87771.2355000001|Público|22855,6206|BUCKLIN|488,2457|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|SHARON SPGS|388,7404|
|TEXAS|123400,5101|Público|13650,9079|AMARILLO|246,2598|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|PERRYTON|289,3178|
|TEXAS|123400,5101|Observador entrenado|13997,7124|CLAUDE|421,44|

Use la opción ' with Others ':

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State with others = "All Other States" by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation with others = "All Other End Locations" by  sum(BeginLat)


```

|State|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|KANSAS|87771.2355000001|Cuerpos de seguridad|18744,823|FT SCOTT|264,858|
|KANSAS|87771.2355000001|Público|22855,6206|BUCKLIN|488,2457|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|SHARON SPGS|388,7404|
|TEXAS|123400,5101|Público|13650,9079|AMARILLO|246,2598|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|PERRYTON|289,3178|
|TEXAS|123400,5101|Observador entrenado|13997,7124|CLAUDE|421,44|
|KANSAS|87771.2355000001|Cuerpos de seguridad|18744,823|Todas las demás ubicaciones de finalización|18479,965|
|KANSAS|87771.2355000001|Público|22855,6206|Todas las demás ubicaciones de finalización|22367,3749|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|Todas las demás ubicaciones de finalización|20890,9679|
|TEXAS|123400,5101|Público|13650,9079|Todas las demás ubicaciones de finalización|13404,6481|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|Todas las demás ubicaciones de finalización|36939,2788|
|TEXAS|123400,5101|Observador entrenado|13997,7124|Todas las demás ubicaciones de finalización|13576,2724|
|KANSAS|87771.2355000001|||Todas las demás ubicaciones de finalización|24891,0836|
|TEXAS|123400,5101|||Todas las demás ubicaciones de finalización|58523.2932000001|
|Todos los demás Estados|1149279,5923|||Todas las demás ubicaciones de finalización|1149279,5923|

En la consulta siguiente se muestran los mismos resultados para el primer nivel utilizado en el ejemplo anterior.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
 StormEvents
 | where State !in ('TEXAS', 'KANSAS')
 | summarize sum(BeginLat)
```

|sum_BeginLat|
|---|
|1149279,5923|


Solicite otra columna (EventType) al resultado de nivel superior anidado.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 1 of EndLocation by sum(BeginLat), top-nested of EventType  by tmp = max(1)
| project-away tmp
```

|State|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|EventType|
|---|---|---|---|---|---|---|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|SHARON SPGS|388,7404|Viento de tormenta|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|SHARON SPGS|388,7404|Granizo|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|SHARON SPGS|388,7404|Tornado|
|KANSAS|87771.2355000001|Público|22855,6206|BUCKLIN|488,2457|Granizo|
|KANSAS|87771.2355000001|Público|22855,6206|BUCKLIN|488,2457|Viento de tormenta|
|KANSAS|87771.2355000001|Público|22855,6206|BUCKLIN|488,2457|Inundación|
|TEXAS|123400,5101|Observador entrenado|13997,7124|CLAUDE|421,44|Granizo|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|PERRYTON|289,3178|Granizo|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|PERRYTON|289,3178|Inundación|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|PERRYTON|289,3178|Riada|

Asigne un criterio de ordenación de índice para cada valor de este nivel (por grupo) para ordenar el resultado por el último nivel anidado (en este ejemplo, por EndLocation):

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State  by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 4 of EndLocation by  sum(BeginLat)
| order by State , Source, aggregated_EndLocation
| summarize EndLocations = make_list(EndLocation, 10000) , endLocationSums = make_list(aggregated_EndLocation, 10000) by State, Source
| extend indicies = range(0, array_length(EndLocations) - 1, 1)
| mv-expand EndLocations, endLocationSums, indicies
```

|State|Source|EndLocations|endLocationSums|índices|
|---|---|---|---|---|
|TEXAS|Observador entrenado|CLAUDE|421,44|0|
|TEXAS|Observador entrenado|AMARILLO|316,8892|1|
|TEXAS|Observador entrenado|DALHART|252,6186|2|
|TEXAS|Observador entrenado|PERRYTON|216,7826|3|
|TEXAS|Cuerpos de seguridad|PERRYTON|289,3178|0|
|TEXAS|Cuerpos de seguridad|LEAKEY|267,9825|1|
|TEXAS|Cuerpos de seguridad|BRACKETTVILLE|264,3483|2|
|TEXAS|Cuerpos de seguridad|GILMER|261,9068|3|
|KANSAS|Observador entrenado|SHARON SPGS|388,7404|0|
|KANSAS|Observador entrenado|ATWOOD|358,6136|1|
|KANSAS|Observador entrenado|LENORA|317,0718|2|
|KANSAS|Observador entrenado|SCOTT CITY|307,84|3|
|KANSAS|Público|BUCKLIN|488,2457|0|
|KANSAS|Público|ASHLAND|446,4218|1|
|KANSAS|Público|FRENTE|446,11|2|
|KANSAS|Público|PARQUE DE ESTADO DE MEADE|371,1|3|

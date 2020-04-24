---
title: 'operador de la parte superior anidada: Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador anidado superior en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 296c36f4653bec971c71dc210008af7b0d98959a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505940"
---
# <a name="top-nested-operator"></a>top-nested operator

genera los principales resultados jerárquicos, donde cada nivel es un desglose según los valores de niveles anteriores. 

```kusto
T | top-nested 3 of Location with others="Others" by sum(MachinesNumber), top-nested 4 of bin(Timestamp,5m) by sum(MachinesNumber)
```

Es útil para escenarios de visualización de paneles, o cuando es necesario responder a una pregunta que suena como: "encontrar cuáles son los valores N superiores de K1 (usando alguna agregación); para cada uno de ellos, encontrar cuáles son los valores SuperiorM de K2 (usando otra agregación); ..."

**Sintaxis**

*T* `|` `desc``,``asc` |  `by` `=``with others=` `of` *Expression1* *Aggregation1* *N1**AggName1* *ConstExpr1*[ N1 ] Expresión1 [ ConstExpr1 ] [ AggName1 ] Aggregation1 [ ] [ ...] `top-nested`

**Argumentos**

para cada regla anidada:
* *N1*: El número de valores superiores que se devolverán para cada nivel de jerarquía. Opcional (si se omite, se devolverán todos los valores distintos).
* *Expresión1*: Una expresión por la que seleccionar los valores superiores. Normalmente es un nombre de columna en *T*o alguna operación `bin()`de binning (por ejemplo, ) en una columna de este tipo. 
* *ConstExpr1*: Si se especifica, para el nivel de anidamiento aplicable, se añadirá una fila adicional que contiene el resultado agregado para los demás valores que no se incluyen en los valores superiores.
* *Agregación1*: Una llamada a una función de agregación que puede ser una de: [sum()](sum-aggfunction.md), [count()](count-aggfunction.md), [max()](max-aggfunction.md), [min()](min-aggfunction.md), [dcount()](dcountif-aggfunction.md), [avg()](avg-aggfunction.md), [percentile()](percentiles-aggfunction.md), [percentilew()](percentiles-aggfunction.md), o cualquier combinación algebric de estas agregaciones.
* `asc` o `desc` (valor predeterminado) puede aparecer para controlar si la selección se hace realmente desde la parte "superior" o desde la parte "inferior" del intervalo.

**Devuelve**

Tabla jerácea que incluye columnas de entrada y para cada una se genera una nueva columna para incluir el resultado de la agregación para el mismo nivel para cada elemento.
Las columnas se organizan en el mismo orden de las columnas de entrada y la nueva columna producida estará cerca de la columna agregada. Cada registro tiene una estructura jerárquica donde se selecciona cada valor después de aplicar todas las reglas anidadas superiores anteriores en todos los niveles anteriores y, a continuación, aplicar la regla del nivel actual en esta salida.
Esto significa que los valores n superiores para el nivel i se calculan para cada valor en el nivel i - 1.
 
**Sugerencias**

* Usar columnas de cambio de nombre en los resultados de *la agregación:* T ? anidado en la parte superior 3 de Ubicación por MachinesNumberForLocation - sum(MachinesNumber) ... .

* El número de registros devueltos podría ser bastante grande; hasta (*N1*+1) \* (*N2*+1) \* ... \* (*Nm*+1) (donde m es el número de los niveles y *Ni* es el recuento superior para el nivel i).

* La agregación debe recibir una columna numérica con la función de agregación que es uno de los mencionados anteriormente.

* Utilice `with others=` la opción para conseguir el valor agregado de todos los otros valores que no fueron valores N superiores en algún nivel.

* Si no está interesado `with others=` en obtener para algún nivel, se anexarán valores nulos (para la columna aggreagated y la clave de nivel, consulte el ejemplo siguiente).


* Es posible devolver columnas adicionales para los candidatos anidados superiores seleccionados añadiendo instrucciones adicionales anidadas superiores como estas (consulte los ejemplos a continuación):

```kusto
top-nested 2 of ...., ..., ..., top-nested of <additionalRequiredColumn1> by max(1), top-nested of <additionalRequiredColumn2> by max(1)
```

**Ejemplo**

```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation by sum(BeginLat)
```

|State|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|Kansas|87771.2355000001|Cuerpos de seguridad|18744.823|FT SCOTT|264.858|
|Kansas|87771.2355000001|Público|22855.6206|Bucklin|488.2457|
|Kansas|87771.2355000001|Observador entrenado|21279.7083|SHARON SPGS|388.7404|
|TEXAS|123400.5101|Público|13650.9079|AMARILLO|246.2598|
|TEXAS|123400.5101|Cuerpos de seguridad|37228.5966|PERRYTON|289.3178|
|TEXAS|123400.5101|Observador entrenado|13997.7124|Claude|421.44|


* Con otro ejemplo:

```kusto
StormEvents
| top-nested 2 of State with others = "All Other States" by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation with others = "All Other End Locations" by  sum(BeginLat)


```

|State|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|Kansas|87771.2355000001|Cuerpos de seguridad|18744.823|FT SCOTT|264.858|
|Kansas|87771.2355000001|Público|22855.6206|Bucklin|488.2457|
|Kansas|87771.2355000001|Observador entrenado|21279.7083|SHARON SPGS|388.7404|
|TEXAS|123400.5101|Público|13650.9079|AMARILLO|246.2598|
|TEXAS|123400.5101|Cuerpos de seguridad|37228.5966|PERRYTON|289.3178|
|TEXAS|123400.5101|Observador entrenado|13997.7124|Claude|421.44|
|Kansas|87771.2355000001|Cuerpos de seguridad|18744.823|Todas las demás ubicaciones finales|18479.965|
|Kansas|87771.2355000001|Público|22855.6206|Todas las demás ubicaciones finales|22367.3749|
|Kansas|87771.2355000001|Observador entrenado|21279.7083|Todas las demás ubicaciones finales|20890.9679|
|TEXAS|123400.5101|Público|13650.9079|Todas las demás ubicaciones finales|13404.6481|
|TEXAS|123400.5101|Cuerpos de seguridad|37228.5966|Todas las demás ubicaciones finales|36939.2788|
|TEXAS|123400.5101|Observador entrenado|13997.7124|Todas las demás ubicaciones finales|13576.2724|
|Kansas|87771.2355000001|||Todas las demás ubicaciones finales|24891.0836|
|TEXAS|123400.5101|||Todas las demás ubicaciones finales|58523.2932000001|
|Todos los demás estados|1149279.5923|||Todas las demás ubicaciones finales|1149279.5923|


La siguiente consulta muestra los mismos resultados para el primer nivel utilizado en el ejemplo anterior:

```kusto
 StormEvents
 | where State !in ('TEXAS', 'KANSAS')
 | summarize sum(BeginLat)
```

|sum_BeginLat|
|---|
|1149279.5923|


Solicitar otra columna (EventType) al resultado anidado superior: 

```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 1 of EndLocation by sum(BeginLat), top-nested of EventType  by tmp = max(1)
| project-away tmp
```

|State|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|EventType|
|---|---|---|---|---|---|---|
|Kansas|87771.2355000001|Observador entrenado|21279.7083|SHARON SPGS|388.7404|Viento de tormenta|
|Kansas|87771.2355000001|Observador entrenado|21279.7083|SHARON SPGS|388.7404|Granizo|
|Kansas|87771.2355000001|Observador entrenado|21279.7083|SHARON SPGS|388.7404|Tornado|
|Kansas|87771.2355000001|Público|22855.6206|Bucklin|488.2457|Granizo|
|Kansas|87771.2355000001|Público|22855.6206|Bucklin|488.2457|Viento de tormenta|
|Kansas|87771.2355000001|Público|22855.6206|Bucklin|488.2457|Inundación|
|TEXAS|123400.5101|Observador entrenado|13997.7124|Claude|421.44|Granizo|
|TEXAS|123400.5101|Cuerpos de seguridad|37228.5966|PERRYTON|289.3178|Granizo|
|TEXAS|123400.5101|Cuerpos de seguridad|37228.5966|PERRYTON|289.3178|Inundación|
|TEXAS|123400.5101|Cuerpos de seguridad|37228.5966|PERRYTON|289.3178|Riada|

Para ordenar el resultado por el último nivel anidado (en este ejemplo por EndLocation) y dar un criterio de ordenación de índice para cada valor en este nivel (por grupo):

```kusto
StormEvents
| top-nested 2 of State  by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 4 of EndLocation by  sum(BeginLat)
| order by State , Source, aggregated_EndLocation
| summarize EndLocations = make_list(EndLocation, 10000) , endLocationSums = make_list(aggregated_EndLocation, 10000) by State, Source
| extend indicies = range(0, array_length(EndLocations) - 1, 1)
| mv-expand EndLocations, endLocationSums, indicies
```

|State|Source|EndLocations|endLocationSums|acusaciones|
|---|---|---|---|---|
|TEXAS|Observador entrenado|Claude|421.44|0|
|TEXAS|Observador entrenado|AMARILLO|316.8892|1|
|TEXAS|Observador entrenado|Dalhart|252.6186|2|
|TEXAS|Observador entrenado|PERRYTON|216.7826|3|
|TEXAS|Cuerpos de seguridad|PERRYTON|289.3178|0|
|TEXAS|Cuerpos de seguridad|Leakey|267.9825|1|
|TEXAS|Cuerpos de seguridad|BRACKETTVILLE|264.3483|2|
|TEXAS|Cuerpos de seguridad|Gilmer|261.9068|3|
|Kansas|Observador entrenado|SHARON SPGS|388.7404|0|
|Kansas|Observador entrenado|Atwood|358.6136|1|
|Kansas|Observador entrenado|Lenora|317.0718|2|
|Kansas|Observador entrenado|SCOTT CITY|307.84|3|
|Kansas|Público|Bucklin|488.2457|0|
|Kansas|Público|Ashland|446.4218|1|
|Kansas|Público|Protección|446.11|2|
|Kansas|Público|PARQUE ESTATAL DE MEADE|371.1|3|
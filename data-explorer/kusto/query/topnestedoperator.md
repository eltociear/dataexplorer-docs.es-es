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
ms.openlocfilehash: f87606442dcc5d7c9e7e0fceec379c37169757c3
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83370768"
---
# <a name="top-nested-operator"></a>top-nested operator

genera los principales resultados jerárquicos, donde cada nivel es un desglose según los valores de niveles anteriores. 

```kusto
T | top-nested 3 of Location with others="Others" by sum(MachinesNumber), top-nested 4 of bin(Timestamp,5m) by sum(MachinesNumber)
```

Es útil para los escenarios de visualización del panel o cuando es necesario responder a una pregunta similar a la siguiente: "buscar los valores Top-N de K1 (usando alguna agregación); para cada uno de ellos, busque Cuáles son los valores Top-M de K2 (mediante otra agregación); ..."

**Sintaxis**

*T* `|` `top-nested` [*N1*] `of` *expresión1* [ `with others=` *ConstExpr1*] `by` [*AggName1* `=` ] *Aggregation1* [ `asc`  |  `desc` ] [ `,` ...]

**Argumentos**

para cada regla anidada superior:
* *N1*: el número de valores superiores que se va a devolver para cada nivel de jerarquía. Opcional (si se omite, se devolverán todos los valores distintos).
* *Expression1*: expresión por la que se van a seleccionar los valores superiores. Normalmente, es un nombre de columna en *T*o alguna operación discretización (por ejemplo, `bin()` ) en una columna de este tipo. 
* *ConstExpr1*: si se especifica, para el nivel de anidamiento aplicable, se anexará una fila adicional que contenga el resultado agregado para los demás valores que no estén incluidos en los valores superiores.
* *Aggregation1*: una llamada a una función de agregación que puede ser una de las siguientes: [SUM ()](sum-aggfunction.md), [Count ()](count-aggfunction.md), [Max ()](max-aggfunction.md), [min ()](min-aggfunction.md), [DCont ()](dcountif-aggfunction.md), [AVG ()](avg-aggfunction.md), [percentil ()](percentiles-aggfunction.md), [percentilew ()](percentiles-aggfunction.md), o cualquier combinación algebric de estas agregaciones.
* `asc` o `desc` (valor predeterminado) puede aparecer para controlar si la selección se hace realmente desde la parte "superior" o desde la parte "inferior" del intervalo.

**Devuelve**

Tabla jerárquica que incluye las columnas de entrada y para cada una de ellas se genera una nueva columna para incluir el resultado de la agregación para el mismo nivel para cada elemento.
Las columnas se organizan en el mismo orden que las columnas de entrada y la nueva columna generada se aproximará a la columna agregada. Cada registro tiene una estructura jerárquica donde cada valor se selecciona después de aplicar todas las reglas principales anidadas en todos los niveles anteriores y, a continuación, aplicar la regla del nivel actual en esta salida.
Esto significa que los n valores superiores para el nivel i se calculan para cada valor en el nivel i-1.
 
**Sugerencias**

* Usar columnas cambiar nombre en para resultados de *agregación* : T | superior-anidado 3 de ubicación por MachinesNumberForLocation = SUM (MachinesNumber)....

* El número de registros devueltos puede ser bastante grande. hasta (*N1*+ 1) \* (*N2*+ 1) \* ... \* (*nm*+ 1) (donde m es el número de niveles y *ni* es el recuento superior del nivel i).

* La agregación debe recibir una columna numérica con la función de agregación, que es uno de los mencionados anteriormente.

* Use la `with others=` opción para obtener el valor agregado de todos los demás valores que no eran valores N superiores en algún nivel.

* Si no le interesa obtener `with others=` algún nivel, se anexarán valores NULL (para la columna aggreagated y la clave de nivel, vea el ejemplo siguiente).


* Es posible devolver columnas adicionales para los candidatos principales anidados, para lo que se deben anexar instrucciones superiores anidadas adicionales como estas (vea los ejemplos siguientes):

```kusto
top-nested 2 of ...., ..., ..., top-nested of <additionalRequiredColumn1> by max(1), top-nested of <additionalRequiredColumn2> by max(1)
```

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation by sum(BeginLat)
```

|Estado|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
|---|---|---|---|---|---|
|KANSAS|87771.2355000001|Cuerpos de seguridad|18744,823|FT SCOTT|264,858|
|KANSAS|87771.2355000001|Público|22855,6206|BUCKLIN|488,2457|
|KANSAS|87771.2355000001|Observador entrenado|21279,7083|SHARON SPGS|388,7404|
|TEXAS|123400,5101|Público|13650,9079|AMARILLO|246,2598|
|TEXAS|123400,5101|Cuerpos de seguridad|37228,5966|PERRYTON|289,3178|
|TEXAS|123400,5101|Observador entrenado|13997,7124|CLAUDE|421,44|


* En otros ejemplos:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State with others = "All Other States" by sum(BeginLat),
  top-nested 3 of Source by sum(BeginLat),
  top-nested 1 of EndLocation with others = "All Other End Locations" by  sum(BeginLat)


```

|Estado|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|
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


La siguiente consulta muestra los mismos resultados para el primer nivel utilizado en el ejemplo anterior:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
 StormEvents
 | where State !in ('TEXAS', 'KANSAS')
 | summarize sum(BeginLat)
```

|sum_BeginLat|
|---|
|1149279,5923|


Solicitar otra columna (EventType) al resultado superior anidado: 

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 1 of EndLocation by sum(BeginLat), top-nested of EventType  by tmp = max(1)
| project-away tmp
```

|Estado|aggregated_State|Source|aggregated_Source|EndLocation|aggregated_EndLocation|EventType|
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

Para ordenar el resultado por el último nivel anidado (en este ejemplo por EndLocation) y proporcionar un criterio de ordenación de índice para cada valor de este nivel (por grupo):

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| top-nested 2 of State  by sum(BeginLat),    top-nested 2 of Source by sum(BeginLat),    top-nested 4 of EndLocation by  sum(BeginLat)
| order by State , Source, aggregated_EndLocation
| summarize EndLocations = make_list(EndLocation, 10000) , endLocationSums = make_list(aggregated_EndLocation, 10000) by State, Source
| extend indicies = range(0, array_length(EndLocations) - 1, 1)
| mv-expand EndLocations, endLocationSums, indicies
```

|Estado|Source|EndLocations|endLocationSums|índices|
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

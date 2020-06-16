---
title: 'complemento dinámico: Azure Explorador de datos'
description: En este artículo se describe el complemento Pivot en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4662b1bd9f68778cab1f799f564499e23add5812
ms.sourcegitcommit: 6a0bd5b84f9bd739510c6a75277dec3a9e851edd
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84788909"
---
# <a name="pivot-plugin"></a>complemento Pivot

Gira una tabla convirtiendo los valores únicos de una columna de la tabla de entrada en varias columnas en la tabla de salida y realiza agregaciones donde son necesarias en cualquier valor de columna restante que se desee en la salida final.

```kusto
T | evaluate pivot(PivotColumn)
```

**Sintaxis**

`T | evaluate pivot(`*pivotColumn* `[, ` *aggregationFunction* `] [,` *Columna1* `[,` *columna2* ...`]])`

**Argumentos**

* *pivotColumn*: la columna que se va a girar. cada valor único de esta columna será una columna de la tabla de salida.
* *función de agregación*: (opcional) agrega varias filas de la tabla de entrada a una sola fila de la tabla de salida. Funciones admitidas actualmente: `min()` , `max()` , `any()` , `sum()` , `dcount()` , `avg()` , `stdev()` , `variance()` , `make_list()` , `make_bag()` , `make_set()` , (el `count()` valor predeterminado es `count()` ).
* *column1*, *columna2*,...: (opcional) nombres de columna. La tabla de salida contendrá una columna adicional por cada columna especificada. valor predeterminado: todas las columnas distintas de la columna dinamizada y la columna de agregación.

**Devuelve**

Pivot devuelve la tabla girada con las columnas especificadas (*column1*, *columna2*,...) y todos los valores únicos de las columnas dinámicas. Cada celda de las columnas dinamizadas contendrá el cálculo de la función de agregado.

**Note**

El esquema de salida del `pivot` complemento se basa en los datos y, por lo tanto, la consulta puede generar un esquema diferente para dos ejecuciones. Esto también significa que la consulta que hace referencia a columnas desempaquetadas puede quedar "rotada" en cualquier momento. Debido a esta razón, no se recomienda usar este complemento para trabajos de automatización.

**Ejemplos**

### <a name="pivot-by-a-column"></a>Dinamizar por columna

Para cada EventType y Estados que empiezan por "AL", cuente el número de eventos de este tipo en este estado.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| project State, EventType 
| where State startswith "AL" 
| where EventType has "Wind" 
| evaluate pivot(State)
```

|EventType|ALABAMA|ALASKA|
|---|---|---|
|Viento de tormenta|352|1|
|Viento alta|0|95|
|Extrema frío/viento refrigerante|0|10|
|Viento fuerte|22|0|


### <a name="pivot-by-a-column-with-aggregation-function"></a>Dinamizar por una columna con la función de agregación.

Para cada EventType y los Estados que empiezan por "AR", se muestra el número total de muertes directas.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State startswith "AR" 
| project State, EventType, DeathsDirect 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect))
```

|EventType|ARKANSAS|ARIZONA|
|---|---|---|
|Lluvia intensa|1|0|
|Viento de tormenta|1|0|
|Vertiginosa|0|1|
|Riada|0|6|
|Viento fuerte|1|0|
|Heat (Calor)|3|0|


### <a name="pivot-by-a-column-with-aggregation-function-and-a-single-additional-column"></a>Dinamizar por una columna con la función de agregación y una sola columna adicional.

El resultado es idéntico al ejemplo anterior.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State startswith "AR" 
| project State, EventType, DeathsDirect 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect), EventType)
```

|EventType|ARKANSAS|ARIZONA|
|---|---|---|
|Lluvia intensa|1|0|
|Viento de tormenta|1|0|
|Vertiginosa|0|1|
|Riada|0|6|
|Viento fuerte|1|0|
|Heat (Calor)|3|0|


### <a name="specify-the-pivoted-column-aggregation-function-and-multiple-additional-columns"></a>Especifique la columna dinamizada, la función de agregación y varias columnas adicionales.

Para cada tipo de evento, origen y estado, suma el número de muertes directas.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State startswith "AR" 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect), EventType, Source)
```

|EventType|Source|ARKANSAS|ARIZONA|
|---|---|---|---|
|Lluvia intensa|Administrador de emergencia|1|0|
|Viento de tormenta|Administrador de emergencia|1|0|
|Vertiginosa|Periódico|0|1|
|Riada|Observador entrenado|0|2|
|Riada|Medios de difusión|0|3|
|Riada|Periódico|0|1|
|Viento fuerte|Cuerpos de seguridad|1|0|
|Heat (Calor)|Periódico|3|0|

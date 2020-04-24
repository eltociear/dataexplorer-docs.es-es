---
title: complemento de pivote - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el complemento dinámico en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e4ec5a94483ade822280ee4a71106c214bb4b9a5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511108"
---
# <a name="pivot-plugin"></a>plugin pivote

Gira una tabla convirtiendo los valores únicos de una columna de la tabla de entrada en varias columnas de la tabla de salida y realiza agregaciones donde se requieren en los valores de columna restantes que se desean en la salida final.

```kusto
T | evaluate pivot(PivotColumn)
```

**Sintaxis**

`T | evaluate pivot(`*pivotColumn*`[, `*aggregationFunction*`] [,`*column1* `[,` *column2* ...`]])`

**Argumentos**

* *pivotColumn*: La columna que se va a rotar. cada valor único de esta columna será una columna en la tabla de salida.
* *Función*de agregación : (opcional) agrega varias filas de la tabla de entrada a una sola fila de la tabla de salida. Funciones admitidas `max()` `any()`actualmente: `avg()` `min()` `stdev()`, `variance()`, `count()` , `sum()` `count()`, `dcount()`, , , , , y (el valor predeterminado es ).
* *column1*, *column2*, ...: (opcional) nombres de columna. La tabla de salida contendrá una columna adicional por cada columna especificada. predeterminado: todas las columnas que no sean la columna pivotada y la columna de agregación.

**Devuelve**

Pivot devuelve la tabla rotada con columnas especificadas (*column1*, *column2*, ...) además de todos los valores únicos de las columnas dinámicas. Cada celda de las columnas pivotadas contendrá el cálculo de la función de agregado.

**Nota**

El esquema de `pivot` salida del complemento se basa en los datos y, por lo tanto, la consulta puede producir un esquema diferente para dos ejecuciones cualquiera. Esto también significa que la consulta que hace referencia a columnas desempaquetadas puede volverse 'rota' en cualquier momento. Debido a esta razón - no se recomienda utilizar este plugin para trabajos de automatización.

**Ejemplos**

### <a name="pivot-by-a-column"></a>Pivote por una columna

Para cada EventType y Estados que comienzan con 'AL', cuente el número de eventos de este tipo en este estado.

```kusto
StormEvents
| project State, EventType 
| where State startswith "AL" 
| where EventType has "Wind" 
| evaluate pivot(State)
```

|EventType|ALABAMA|Alaska|
|---|---|---|
|Viento de tormenta|352|1|
|Viento alto|0|95|
|Frío extremo/viento frío|0|10|
|Viento fuerte|22|0|


### <a name="pivot-by-a-column-with-aggregation-function"></a>Pivote por una columna con función de agregación.

Para cada EventType y Estados que comienzan con 'AR', muestre el número total de muertes directas.

```kusto
StormEvents 
| where State startswith "AR" 
| project State, EventType, DeathsDirect 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect))
```

|EventType|Arkansas|Arizona|
|---|---|---|
|Lluvia fuerte|1|0|
|Viento de tormenta|1|0|
|Relámpago|0|1|
|Riada|0|6|
|Viento fuerte|1|0|
|Heat (Calor)|3|0|


### <a name="pivot-by-a-column-with-aggregation-function-and-a-single-additional-column"></a>Pivote por una columna con función de agregación y una sola columna adicional.

El resultado es idéntico al ejemplo anterior.

```kusto
StormEvents 
| where State startswith "AR" 
| project State, EventType, DeathsDirect 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect), EventType)
```

|EventType|Arkansas|Arizona|
|---|---|---|
|Lluvia fuerte|1|0|
|Viento de tormenta|1|0|
|Relámpago|0|1|
|Riada|0|6|
|Viento fuerte|1|0|
|Heat (Calor)|3|0|


### <a name="specify-the-pivoted-column-aggregation-function-and-multiple-additional-columns"></a>Especifique la columna pivotada, la función de agregación y varias columnas adicionales.

Para cada tipo de evento, fuente y estado, sume el número de muertes directas.

```kusto
StormEvents 
| where State startswith "AR" 
| where DeathsDirect > 0
| evaluate pivot(State, sum(DeathsDirect), EventType, Source)
```

|EventType|Source|Arkansas|Arizona|
|---|---|---|---|
|Lluvia fuerte|Administrador de emergencia|1|0|
|Viento de tormenta|Administrador de emergencia|1|0|
|Relámpago|Periódico|0|1|
|Riada|Observador entrenado|0|2|
|Riada|Medios de difusión|0|3|
|Riada|Periódico|0|1|
|Viento fuerte|Cuerpos de seguridad|1|0|
|Heat (Calor)|Periódico|3|0|
---
title: rolling_percentile complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe rolling_percentile complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 02def4069c83eeec080ca059493132619fce30d5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510275"
---
# <a name="rolling_percentile-plugin"></a>rolling_percentile plugin

Devuelve una estimación para el percentil especificado de la población *ValueColumn* en una ventana de tamaño *BinsPerWindow* móvil (deslizante) por *BinSize*.

```kusto
T | evaluate rolling_percentile(ValueColumn, Percentile, IndexColumn, BinSize, BinsPerWindow)
```

**Sintaxis**

*T* `| evaluate` `,` `,` `,` `,` `,` *ValueColumn* `,` `,` *BinSize* *IndexColumn* *dim2* *dim1* *Percentile* *BinsPerWindow* ValueColumn Percentile IndexColumn BinSize BinsPerWindow [ dim1 dim2 ...] `rolling_percentile(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *ValueColumn*: El nombre de la columna con valores para calcular el percentil de. 
* *Percentil*: Escalar con el percentil a calcular.
* *IndexColumn*: el nombre de la columna para ejecutar la ventana de desplazamiento.
* *BinSize*: Escalar con el tamaño de las ubicaciones que se aplicarán sobre *IndexColumn*.
* *BinsPerWindow*: Escalar con número de ubicaciones incluidas en cada ventana.
* *dim1*, *dim2*, ... : (opcional) lista de las columnas de dimensiones para cortar.

**Devuelve**

Devuelve una tabla con una fila por cada ubicación (y combinación de dimensiones si se especifica) que tiene el percentil móvil de valores en la ventana que termina en la ubicación (incluido). valores de recuento distintos, recuento distinto de nuevos valores, recuento distinto agregado para cada ventana de tiempo.

El esquema de la tabla de salida es:


|IndexColumn|dim1|...|dim_n|rolling_BinsPerWindow_percentile_ValueColumn_Pct
|---|---|---|---|---|


**Ejemplos**

### <a name="rolling-3-day-median-value-per-day"></a>Rodar valor medio de 3 días por día 

La siguiente consulta calcula un valor mediano de 3 días en granularidad diaria. Cada fila de la salida representa el valor mediano de los últimos 3 bins (días), incluida la propia ubicación.

```kusto
let T = 
range idx from 0 to 24*10-1 step 1
| project Timestamp = datetime(2018-01-01) + 1h*idx, val=idx+1
| extend EvenOrOdd = iff(val % 2 == 0, "Even", "Odd");
 T  
 | evaluate rolling_percentile(val, 50, Timestamp, 1d, 3)
```

|Timestamp|rolling_3_percentile_val_50|
|---|---|
|2018-01-01 00:00:00.0000000|   12|
|2018-01-02 00:00:00.0000000|   24|
|2018-01-03 00:00:00.0000000|   36|
|2018-01-04 00:00:00.0000000|   60|
|2018-01-05 00:00:00.0000000|   84|
|2018-01-06 00:00:00.0000000|   108|
|2018-01-07 00:00:00.0000000|   132|
|2018-01-08 00:00:00.0000000|   156|
|2018-01-09 00:00:00.0000000|   180|
|2018-01-10 00:00:00.0000000|   204|

### <a name="rolling-3-day-median-value-per-day-by-dimension"></a>Rodar valor medio de 3 días por día por dimensión

El mismo ejemplo anterior, pero ahora también calcula la ventana móvil particionada para cada valor de la dimensión.

```kusto
let T = 
range idx from 0 to 24*10-1 step 1
| project Timestamp = datetime(2018-01-01) + 1h*idx, val=idx+1
| extend EvenOrOdd = iff(val % 2 == 0, "Even", "Odd");
 T  
 | evaluate rolling_percentile(val, 50, Timestamp, 1d, 3, EvenOrOdd)
```

|Timestamp| EvenOrOdd|  rolling_3_percentile_val_50|
|---|---|---|
|2018-01-01 00:00:00.0000000|   Incluso|   12|
|2018-01-02 00:00:00.0000000|   Incluso|   24|
|2018-01-03 00:00:00.0000000|   Incluso|   36|
|2018-01-04 00:00:00.0000000|   Incluso|   60|
|2018-01-05 00:00:00.0000000|   Incluso|   84|
|2018-01-06 00:00:00.0000000|   Incluso|   108|
|2018-01-07 00:00:00.0000000|   Incluso|   132|
|2018-01-08 00:00:00.0000000|   Incluso|   156|
|2018-01-09 00:00:00.0000000|   Incluso|   180|
|2018-01-10 00:00:00.0000000|   Incluso|   204|
|2018-01-01 00:00:00.0000000|   Extraño|    11|
|2018-01-02 00:00:00.0000000|   Extraño|    23|
|2018-01-03 00:00:00.0000000|   Extraño|    35|
|2018-01-04 00:00:00.0000000|   Extraño|    59|
|2018-01-05 00:00:00.0000000|   Extraño|    83|
|2018-01-06 00:00:00.0000000|   Extraño|    107|
|2018-01-07 00:00:00.0000000|   Extraño|    131|
|2018-01-08 00:00:00.0000000|   Extraño|    155|
|2018-01-09 00:00:00.0000000|   Extraño|    179|
|2018-01-10 00:00:00.0000000|   Extraño|    203|
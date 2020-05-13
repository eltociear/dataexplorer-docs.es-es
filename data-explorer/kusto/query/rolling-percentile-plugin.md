---
title: 'complemento de rolling_percentile: Azure Explorador de datos'
description: En este artículo se describe rolling_percentile complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a41a45fb12fafe62fffd6c13e5ea9ecff55bb355
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373025"
---
# <a name="rolling_percentile-plugin"></a>complemento de rolling_percentile

Devuelve una estimación para el percentil especificado del rellenado de *ValueColumn* en una ventana de tamaño *BinsPerWindow* graduada (deslizante) por cada *compartimiento*.

```kusto
T | evaluate rolling_percentile(ValueColumn, Percentile, IndexColumn, BinSize, BinsPerWindow)
```

**Sintaxis**

*T* `| evaluate` `rolling_percentile(` *ValueColumn* `,` *percentil* `,` *IndexColumn* `,` *BinSize* `,` *BinsPerWindow* [ `,` *DIM1* `,` *dim2* `,` ...]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *ValueColumn*: nombre de la columna con valores cuyo percentil se va a calcular. 
* *Percentil*: escalar con el percentil que se va a calcular.
* *IndexColumn*: nombre de la columna en la que se va a ejecutar la ventana desplazada.
* *Bining*: escalar con el tamaño de las bandejas que se van a aplicar sobre el *IndexColumn*.
* *BinsPerWindow*: escalar con el número de ubicaciones incluidas en cada ventana.
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones por las que segmentar.

**Devuelve**

Devuelve una tabla con una fila por cada bin (y combinación de dimensiones si se especifica) que tiene el percentil rodante de los valores de la ventana que finaliza en la ubicación (inclusivo). valores de recuento distintivos, recuento distintivo de nuevos valores, recuento distintivo agregado para cada ventana de tiempo.

El esquema de la tabla de salida es:


|IndexColumn|DIM1|...|dim_n|rolling_BinsPerWindow_percentile_ValueColumn_Pct
|---|---|---|---|---|


**Ejemplos**

### <a name="rolling-3-day-median-value-per-day"></a>Valor de mediana de 3 días rodante por día 

La consulta siguiente calcula un valor de mediana de 3 días en la granularidad diaria. Cada fila de la salida representa el valor medio de las últimas 3 bandejas (días), incluido el propio bin.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

### <a name="rolling-3-day-median-value-per-day-by-dimension"></a>Valor de mediana de 3 días gradual por dimensión

El mismo ejemplo anterior, pero ahora calcula también la ventana gradual con particiones para cada valor de la dimensión.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
|2018-01-01 00:00:00.0000000|   Ni|   12|
|2018-01-02 00:00:00.0000000|   Ni|   24|
|2018-01-03 00:00:00.0000000|   Ni|   36|
|2018-01-04 00:00:00.0000000|   Ni|   60|
|2018-01-05 00:00:00.0000000|   Ni|   84|
|2018-01-06 00:00:00.0000000|   Ni|   108|
|2018-01-07 00:00:00.0000000|   Ni|   132|
|2018-01-08 00:00:00.0000000|   Ni|   156|
|2018-01-09 00:00:00.0000000|   Ni|   180|
|2018-01-10 00:00:00.0000000|   Ni|   204|
|2018-01-01 00:00:00.0000000|   Ajuste|    11|
|2018-01-02 00:00:00.0000000|   Ajuste|    23|
|2018-01-03 00:00:00.0000000|   Ajuste|    35|
|2018-01-04 00:00:00.0000000|   Ajuste|    59|
|2018-01-05 00:00:00.0000000|   Ajuste|    83|
|2018-01-06 00:00:00.0000000|   Ajuste|    107|
|2018-01-07 00:00:00.0000000|   Ajuste|    131|
|2018-01-08 00:00:00.0000000|   Ajuste|    155|
|2018-01-09 00:00:00.0000000|   Ajuste|    179|
|2018-01-10 00:00:00.0000000|   Ajuste|    203|

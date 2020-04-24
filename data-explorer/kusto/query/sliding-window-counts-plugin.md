---
title: sliding_window_counts complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe sliding_window_counts complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: feab3d0e8f548817be12f202eb2d494bd65aa133
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507504"
---
# <a name="sliding_window_counts-plugin"></a>sliding_window_counts plugin

Calcula recuentos y recuentos distintos de valores en una ventana deslizante durante un período de búsqueda, utilizando la técnica descrita [aquí.](samples.md#performing-aggregations-over-a-sliding-window)

Por ejemplo, para cada *día,* calcule el recuento y el recuento distinto de usuarios en la *semana*anterior. 

```kusto
T | evaluate sliding_window_counts(id, datetime_column, startofday(ago(30d)), startofday(now()), 7d, 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `,` *Start* `,` *End* `,` `,` *Bin* `,` `,` *dim2* `,` *IdColumn* `,` *TimelineColumn* *dim1* *LookbackWindow* IdColumn TimelineColumn Start End LookbackWindow Bin [ dim1 dim2 ...] `sliding_window_counts(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColumn*: el nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: el nombre de la columna que representa la línea de tiempo.
* *Inicio*: Escalar con el valor del período de inicio del análisis.
* *Final*: Escalar con el valor del período final del análisis.
* *LookbackWindow*: Valor constante escalar del período de búsqueda (por ejemplo, para usuarios dcount en el pasado 7d: LookbackWindow a 7d)
* *Bin*: Valor constanct escalar del período de paso del análisis. Puede ser un valor numérico/datetime/timestamp, o una `week` / `month` / `year`cadena que sea uno de , en cuyo caso todos los períodos serán [el inicio del](startofweekfunction.md)/[mes](startofmonthfunction.md)/de la semana en[consecuencia.](startofyearfunction.md) 
* *dim1*, *dim2*, ...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene los valores de recuento y recuento distintos de identificadores en el período de búsqueda, para cada período de escala de tiempo (por bin) y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|dim1|..|dim_n|Count|Dcount|
|---|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|..|..|..|long|long|


**Ejemplos**

Calcular recuentos y recuentos para los usuarios de la semana pasada, para cada día en el período de análisis. 

```kusto
let start = datetime(2017 - 08 - 01);
let end = datetime(2017 - 08 - 07); 
let lookbackWindow = 3d;  
let bin = 1d;
let T = datatable(UserId:string, Timestamp:datetime)
[
'Bob',      datetime(2017 - 08 - 01), 
'David',    datetime(2017 - 08 - 01), 
'David',    datetime(2017 - 08 - 01), 
'John',     datetime(2017 - 08 - 01), 
'Bob',      datetime(2017 - 08 - 01), 
'Ananda',   datetime(2017 - 08 - 02),  
'Atul',     datetime(2017 - 08 - 02), 
'John',     datetime(2017 - 08 - 02), 
'Ananda',   datetime(2017 - 08 - 03), 
'Atul',     datetime(2017 - 08 - 03), 
'Atul',     datetime(2017 - 08 - 03), 
'John',     datetime(2017 - 08 - 03), 
'Bob',      datetime(2017 - 08 - 03), 
'Betsy',    datetime(2017 - 08 - 04), 
'Bob',      datetime(2017 - 08 - 05), 
];
T | evaluate sliding_window_counts(UserId, Timestamp, start, end, lookbackWindow, bin)


```

|Timestamp|Count|Dcount|
|---|---|---|
|2017-08-01 00:00:00.0000000|5|3|
|2017-08-02 00:00:00.0000000|8|5|
|2017-08-03 00:00:00.0000000|13|5|
|2017-08-04 00:00:00.0000000|9|5|
|2017-08-05 00:00:00.0000000|7|5|
|2017-08-06 00:00:00.0000000|2|2|
|2017-08-07 00:00:00.0000000|1|1|
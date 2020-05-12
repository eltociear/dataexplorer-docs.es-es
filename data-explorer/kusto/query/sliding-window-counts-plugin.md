---
title: 'complemento de sliding_window_counts: Azure Explorador de datos'
description: En este artículo se describe sliding_window_counts complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2fbc870eafc45c8c63bea98a64f492d161af4c9b
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226353"
---
# <a name="sliding_window_counts-plugin"></a>complemento de sliding_window_counts

Calcula los recuentos y el recuento distinto de valores en una ventana deslizante durante un período de lookback, mediante la técnica que se describe [aquí](samples.md#performing-aggregations-over-a-sliding-window).

Por ejemplo, para cada *día*, calcula el recuento y el recuento distinto de usuarios de la *semana*anterior. 

```kusto
T | evaluate sliding_window_counts(id, datetime_column, startofday(ago(30d)), startofday(now()), 7d, 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `sliding_window_counts(` *IdColumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *LookbackWindow* `,` *bin* `,` [*DIM1* `,` *dim2* `,` ...]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: escalar con el valor del período de inicio del análisis.
* *End*: escalar con el valor del período de finalización del análisis.
* *LookbackWindow*: valor constante escalar del período lookback (por ejemplo, para `dcount` los usuarios de los últimos 7D: LookbackWindow = 7D)
* *Bin*: valor constante escalar del período de paso de análisis. Este valor puede ser un valor numérico, de fecha y hora o de marca de tiempo. Si el valor es una cadena con el formato `week` / `month` / `year` , todos los puntos serán [iniciodelasemana](startofweekfunction.md) / [startofmonth](startofmonthfunction.md) / [startofyear](startofyearfunction.md). 
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de las métricas de actividad.

**Devuelve**

Devuelve una tabla con los valores Count y DISTINCT Count de los identificadores en el período lookback, para cada período de la escala de tiempo (por bin) y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|`dim1`|..|`dim_n`|`count`|`dcount`|
|---|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|..|..|..|long|long|


**Ejemplos**

Calcula los recuentos y `dcounts` los usuarios de la semana anterior, para cada día del período de análisis. 

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

|Timestamp|Count|`dcount`|
|---|---|---|
|2017-08-01 00:00:00.0000000|5|3|
|2017-08-02 00:00:00.0000000|8|5|
|2017-08-03 00:00:00.0000000|13|5|
|2017-08-04 00:00:00.0000000|9|5|
|2017-08-05 00:00:00.0000000|7|5|
|2017-08-06 00:00:00.0000000|2|2|
|2017-08-07 00:00:00.0000000|1|1|
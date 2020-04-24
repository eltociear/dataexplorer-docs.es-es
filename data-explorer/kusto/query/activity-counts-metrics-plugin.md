---
title: activity_counts_metrics complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe activity_counts_metrics complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e55cd940850440499d227082f57769e499e6a551
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519319"
---
# <a name="activity_counts_metrics-plugin"></a>complemento activity_counts_metrics

Calcula métricas de actividad útiles (valores de recuento total, valores de recuento distintos, recuento distinto de nuevos valores, recuento diferenciado agregado) para cada ventana de tiempo comparadas/agregadas con/con *todas las* ventanas de tiempo anteriores (a diferencia de [activity_metrics complemento](activity-metrics-plugin.md) en el que cada ventana de tiempo se compara con su ventana de tiempo anterior solamente).

```kusto
T | evaluate activity_counts_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `,` *End* `,` *Window* `,` `,` `,` `,` *IdColumn* `,` `,` *Start* *dim2* *TimelineColumn* *dim1* *Cohort*IdColumn TimelineColumn Ventana de inicio de columna [ Cohorte ] [ dim1 dim2 ...] `activity_counts_metrics(` [`,` *Mirar hacia atrás*]`)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColumn*: el nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: el nombre de la columna que representa la línea de tiempo.
* *Inicio*: Escalar con el valor del período de inicio del análisis.
* *Final*: Escalar con el valor del período final del análisis.
* *Ventana*: Escalar con el valor del período de la ventana de análisis. Puede ser un valor numérico/datetime/timestamp, o una `week` / `month` / `year`cadena que sea uno de , en cuyo caso todos los períodos serán [el inicio del](startofweekfunction.md)/[mes](startofmonthfunction.md)/de la semana en[consecuencia.](startofyearfunction.md) 
* *dim1*, *dim2*, ...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene los valores de recuento total, valores de recuento distintos, recuento distinto de nuevos valores, recuento distinto agregado para cada ventana de tiempo.

El esquema de la tabla de salida es:

|TimelineColumn|dim1|...|dim_n|count|dcount|new_dcount|aggregated_dcount
|---|---|---|---|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|..|..|..|long|long|long|long|long


* *TimelineColumn*: La hora de inicio de la ventana de tiempo.
* *recuento*: El recuento total de registros en la ventana de tiempo y *dim(s)*
* *dcount*: Los valores de ID distintos cuentan en la ventana de tiempo y *dim(s)*
* *new_dcount*: Los valores de ID distintos en la ventana de tiempo y *dim(s)* en comparación con todas las ventanas de tiempo anteriores. 
* *aggregated_dcount*: El total de valores de ID distintos agregados de *dim(s)* desde la 1a ventana de tiempo hasta la actual (incluida).

**Ejemplos**

### <a name="daily-activity-counts"></a>La actividad diaria cuenta 

La siguiente consulta calcula los recuentos de actividad diaria para la tabla de entrada proporcionada

```kusto
let start=datetime(2017-08-01);
let end=datetime(2017-08-04);
let window=1d;
let T = datatable(UserId:string, Timestamp:datetime)
[
'A', datetime(2017-08-01),
'D', datetime(2017-08-01), 
'J', datetime(2017-08-01),
'B', datetime(2017-08-01),
'C', datetime(2017-08-02),  
'T', datetime(2017-08-02),
'J', datetime(2017-08-02),
'H', datetime(2017-08-03),
'T', datetime(2017-08-03),
'T', datetime(2017-08-03),
'J', datetime(2017-08-03),
'B', datetime(2017-08-03),
'S', datetime(2017-08-03),
'S', datetime(2017-08-04),
];
 T 
 | evaluate activity_counts_metrics(UserId, Timestamp, start, end, window)
```

|Timestamp|count|dcount|new_dcount|aggregated_dcount|
|---|---|---|---|---|
|2017-08-01 00:00:00.0000000|4|4|4|4|
|2017-08-02 00:00:00.0000000|3|3|2|6|
|2017-08-03 00:00:00.0000000|6|5|2|8|
|2017-08-04 00:00:00.0000000|1|1|0|8|



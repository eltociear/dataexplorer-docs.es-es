---
title: 'complemento de activity_counts_metrics: Azure Explorador de datos'
description: En este artículo se describe activity_counts_metrics complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b06b1c137552ba19f9b1ef5367a25bb72eea5c93
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82742045"
---
# <a name="activity_counts_metrics-plugin"></a>complemento activity_counts_metrics

Calcula métricas de actividad útiles para cada ventana de tiempo comparada o agregada a *todas las* ventanas de tiempo anteriores. Las métricas incluyen: valores de recuento total, valores de recuento distintivos, recuento distintivo de nuevos valores y recuento distintivo agregado. Compare este complemento con [activity_metrics complemento](activity-metrics-plugin.md), en el que cada ventana de tiempo se compara solo con la ventana de tiempo anterior.

```kusto
T | evaluate activity_counts_metrics(id, datetime_column, startofday(ago(30d)), startofday(now()), 1d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `,` *TimelineColumn* `,` *dim2* *Start* `,` *dim1* *IdColumn* `,` *Window* *End* `,` *Cohort*IdColumn TimelineColumn`,` Start`,` end Window [`,` cohorte] [DIM1 dim2...] `| evaluate` `activity_counts_metrics(` [`,` *Lookback*]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: escalar con el valor del período de inicio del análisis.
* *End*: escalar con el valor del período de finalización del análisis.
* *Window*: escalar con el valor del período de la ventana de análisis. Puede ser un valor numérico, de fecha y hora o de marca de tiempo, o una `week` / `month` / `year`cadena que sea uno de, en cuyo caso todos los puntos serán [iniciodelasemana](startofweekfunction.md)/[startofmonth](startofmonthfunction.md) o [startofyear](startofyearfunction.md). 
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de las métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene: valores de recuento totales, valores de recuento distintivos, recuento distinto de nuevos valores y recuento agregado distintivo para cada ventana de tiempo.

El esquema de la tabla de salida es:

|`TimelineColumn`|`dim1`|...|`dim_n`|`count`|`dcount`|`new_dcount`|`aggregated_dcount`
|---|---|---|---|---|---|---|---|---|
|tipo: a partir de*`TimelineColumn`*|..|..|..|long|long|long|long|long


* *`TimelineColumn`*: Hora de inicio de la ventana de tiempo.
* *`count`*: El recuento total de registros en la ventana de tiempo y *Dim (s)*
* *`dcount`*: El recuento de valores de ID. distintivos en la ventana de tiempo y *Dim (s)*
* *`new_dcount`*: Los distintos valores de identificador en la ventana de tiempo y las *cotas* se comparan con todas las ventanas de tiempo anteriores. 
* *`aggregated_dcount`*: El total de valores de ID. distintos agregados de *Dim (s)* desde el primer período de tiempo al actual (inclusivo).

**Ejemplos**

### <a name="daily-activity-counts"></a>Recuentos de actividades diarias 

En la consulta siguiente se calculan los recuentos de actividades diarias de la tabla de entrada proporcionada.

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

|`Timestamp`|`count`|`dcount`|`new_dcount`|`aggregated_dcount`|
|---|---|---|---|---|
|2017-08-01 00:00:00.0000000|4|4|4|4|
|2017-08-02 00:00:00.0000000|3|3|2|6|
|2017-08-03 00:00:00.0000000|6|5|2|8|
|2017-08-04 00:00:00.0000000|1|1|0|8|



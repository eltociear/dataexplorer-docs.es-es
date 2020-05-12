---
title: 'complemento de activity_engagement: Azure Explorador de datos'
description: En este artículo se describe activity_engagement complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9aa85bcb12cd5f8d836f58ea9d16a318d8a40506
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225962"
---
# <a name="activity_engagement-plugin"></a>complemento activity_engagement

calcula la proporción de interacción de actividad según la columna de identificador a través de una ventana deslizante de escala de tiempo.

activity_engagement complemento se puede usar para calcular DAU/WAU/MAU (actividades diarias, semanales y mensuales).

```kusto
T | evaluate activity_engagement(id, datetime_column, 1d, 30d)
```

**Sintaxis**

*T* `| evaluate` `activity_engagement(` *IdColumn* `,` *TimelineColumn* `,` [*Start* `,` *End* `,` ] *InnerActivityWindow* `,` *OuterActivityWindow* [ `,` *DIM1* `,` *dim2* `,` ...]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: (opcional) escalar con el valor del período de inicio del análisis.
* *End*: (opcional) escalar con el valor del período de finalización del análisis.
* *InnerActivityWindow*: escalar con el valor del período de la ventana de análisis del ámbito interno.
* *OuterActivityWindow*: escalar con el valor del período de la ventana de análisis de ámbito externo.
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de las métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene (recuento distintivo de valores de identificador dentro de la ventana de ámbito interno, recuento distinto de valores de identificador dentro de la ventana de ámbito externo y la relación de actividad) para cada período de la ventana de ámbito interno y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|TimelineColumn|dcount_activities_inner|dcount_activities_outer|activity_ratio|DIM1|..|dim_n|
|---|---|---|---|--|--|--|--|--|--|
|tipo: a partir de *TimelineColumn*|long|long|double|..|..|..|


**Ejemplos**

### <a name="dauwau-calculation"></a>Cálculo de DAU/WAU

En el ejemplo siguiente se calcula la relación entre DAU/WAU (usuarios activos diarios y usuarios activos semanales) sobre los datos generados de forma aleatoria.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-01-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Calculate DAU/WAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 7d)
| project _day, Dau_Wau=activity_ratio*100 
| render timechart 
```

:::image type="content" source="images/activity-engagement-plugin/activity-engagement-dau-wau.png" border="false" alt-text="Contrato de actividad Dau Wau":::

### <a name="daumau-calculation"></a>Cálculo de DAU/MAU

En el ejemplo siguiente se calcula la relación entre DAU/WAU (usuarios activos diarios y usuarios activos semanales) sobre los datos generados de forma aleatoria.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
// Calculate DAU/MAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 30d)
| project _day, Dau_Mau=activity_ratio*100 
| render timechart 
```

:::image type="content" source="images/activity-engagement-plugin/activity-engagement-dau-mau.png" border="false" alt-text="Acuerdo de actividad Dau Mau":::

### <a name="daumau-calculation-with-additional-dimensions"></a>Cálculo de DAU/MAU con dimensiones adicionales

En el ejemplo siguiente se calcula la relación entre DAU/WAU (usuarios activos diarios y usuarios activos semanales) con respecto a los datos generados aleatoriamente con dimensiones adicionales ( `mod3` ).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-05-31);
range _day from _start to _end  step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1) 
| mv-expand id=_users to typeof(long) limit 1000000
| extend mod3 = strcat("mod3=", id % 3)
// Calculate DAU/MAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 30d, mod3)
| project _day, Dau_Mau=activity_ratio*100, mod3 
| render timechart 
```

:::image type="content" source="images/activity-engagement-plugin/activity-engagement-dau-mau-mod3.png" border="false" alt-text="Activity Engagement Dau Mau mod 3":::

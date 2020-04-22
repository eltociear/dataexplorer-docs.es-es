---
title: activity_engagement complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe activity_engagement complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: fe6d3f68f59ae52c96d3e9467cc364d792b13cf3
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81664026"
---
# <a name="activity_engagement-plugin"></a>complemento activity_engagement

calcula la proporción de interacción de actividad según la columna de identificador a través de una ventana deslizante de escala de tiempo.

activity_engagement plugin se puede utilizar para calcular DAU /WAU/MAU (actividades diarias / semanales / mensuales).

```kusto
T | evaluate activity_engagement(id, datetime_column, 1d, 30d)
```

**Sintaxis**

*T* `| evaluate` `,` *End*`,``,` `,` `,` `,` *IdColumn* `,` `,` *Start* *dim2* *TimelineColumn* *dim1* *OuterActivityWindow* *InnerActivityWindow* IdColumn TimelineColumn [ Start End ] InnerActivityWindow OuterActivityWindow [ dim1 dim2 ...] `activity_engagement(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColumn*: el nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: el nombre de la columna que representa la línea de tiempo.
* *Inicio*: (opcional) Escalar con el valor del período de inicio del análisis.
* *Final*: (opcional) Escalar con el valor del período final del análisis.
* *InnerActivityWindow*: Escalar con el valor del período de la ventana de análisis de ámbito interno.
* *OuterActivityWindow*: Escalar con el valor del período de la ventana de análisis de ámbito externo.
* *dim1*, *dim2*, ...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene (recuento distinto de valores de identificador dentro de la ventana de ámbito interno, recuento distinto de valores de identificador dentro de la ventana de ámbito externo y la relación de actividad) para cada período de ventana de ámbito interno y para cada combinación de dimensiones existentes.

El esquema de la tabla de salida es:

|TimelineColumn|dcount_activities_inner|dcount_activities_outer|activity_ratio|dim1|..|dim_n|
|---|---|---|---|--|--|--|--|--|--|
|tipo: a partir de *TimelineColumn*|long|long|double|..|..|..|


**Ejemplos**

### <a name="dauwau-calculation"></a>Cálculo DAU/WAU

En el ejemplo siguiente se calcula DAU/WAU (relación de usuarios activos diarios /usuarios activos semanales) en un archivo de datos generados aleatoriamente.

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

:::image type="content" source="images/queries/activity-engagement-dau-wau.png" border="false" alt-text="Compromiso de actividad dau wau":::

### <a name="daumau-calculation"></a>Cálculo DAU/MAU

En el ejemplo siguiente se calcula DAU/WAU (relación de usuarios activos diarios /usuarios activos semanales) en un archivo de datos generados aleatoriamente.

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

:::image type="content" source="images/queries/activity-engagement-dau-mau.png" border="false" alt-text="Compromiso de actividad dau mau":::

### <a name="daumau-calculation-with-additional-dimensions"></a>Cálculo DAU/MAU con dimensiones adicionales

En el ejemplo siguiente se calcula DAU/WAU (relación de usuarios activos diarios/usuarios activos semanales) sobre un dato generado aleatoriamente con dimensión adicional (`mod3`).

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

:::image type="content" source="images/queries/activity-engagement-dau-mau-mod3.png" border="false" alt-text="Compromiso de actividad dau mau mod 3":::

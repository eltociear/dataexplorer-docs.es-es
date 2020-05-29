---
title: 'complemento de active_users_count: Azure Explorador de datos'
description: En este artículo se describe active_users_count complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b40ca669df7671b1451166f6bfc1c7c680713166
ms.sourcegitcommit: 1f50c6688a2b8d8a3976c0cd0ef40cde2ef76749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2020
ms.locfileid: "84202966"
---
# <a name="active_users_count-plugin"></a>complemento de active_users_count

Calcula el recuento distintivo de valores, donde cada valor ha aparecido en al menos un número mínimo de períodos en un período de lookback.

Útil para calcular recuentos distintivos de "ventiladores" solo, sin incluir los aspectos de "no". Un usuario se cuenta como un "ventilador" solo si estaba activo durante el período de lookback. El período de lookback solo se usa para determinar si un usuario se considera `active` ("ventilador") o no. La agregación en sí no incluye a los usuarios de la ventana lookback. En comparación, la agregación de [sliding_window_counts](sliding-window-counts-plugin.md) se realiza en una ventana deslizante del período lookback.

```kusto
T | evaluate active_users_count(id, datetime_column, startofday(ago(30d)), startofday(now()), 7d, 1d, 2, 7d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `active_users_count(` *IdColumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *LookbackWindow* `,` *punto* `,` *ActivePeriodsCount* `,` *bin* `,` [*DIM1* `,` *dim2* `,` ...]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: (opcional) escalar con el valor del período de inicio del análisis.
* *End*: (opcional) escalar con el valor del período de finalización del análisis.
* *LookbackWindow*: ventana de tiempo deslizante que define un punto en el que se comprueba el aspecto del usuario. El período de lookback comienza en ([apariencia actual]-[ventana de lookback]) y termina en ([apariencia actual]). 
* *Period*: intervalo de tiempo constante escalar para contar como apariencia única (un usuario se contará como activo si aparece en al menos un ActivePeriodsCount distinto de este intervalo de tiempo.
* *ActivePeriodsCount*: número mínimo de períodos activos distintos para decidir si el usuario está activo. Los usuarios activos son aquellos que presentaron en al menos (igual o mayor que) los períodos activos.
* *Bin*: valor constante escalar del período de paso de análisis. Puede ser un valor numérico/DateTime/timestamp, o una cadena que sea `week` / `month` / `year` . Todos los puntos serán las funciones [iniciodelasemana](startofweekfunction.md) / [startofmonth](startofmonthfunction.md) / [startofyear](startofyearfunction.md) correspondientes.
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de las métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene valores de recuento distintivos para los identificadores que han aparecido en ActivePeriodCounts en los períodos siguientes: el período lookback, cada período de la escala de tiempo y cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|DIM1|..|dim_n|dcount_values|
|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|..|..|..|long|


**Ejemplos**

Calcular el número semanal de usuarios distintos que aparecían en al menos tres días diferentes durante un período de ocho días antes. Período de análisis: 2018 de julio.

```kusto
let Start = datetime(2018-07-01);
let End = datetime(2018-07-31);
let LookbackWindow = 8d;
let Period = 1d;
let ActivePeriods = 3;
let Bin = 7d; 
let T =  datatable(User:string, Timestamp:datetime)
[
    "B",      datetime(2018-06-29),
    "B",      datetime(2018-06-30),
    "A",      datetime(2018-07-02),
    "B",      datetime(2018-07-04),
    "B",      datetime(2018-07-08),
    "A",      datetime(2018-07-10),
    "A",      datetime(2018-07-14),
    "A",      datetime(2018-07-17),
    "A",      datetime(2018-07-20),
    "B",      datetime(2018-07-24)
]; 
T | evaluate active_users_count(User, Timestamp, Start, End, LookbackWindow, Period, ActivePeriods, Bin)



```

|Timestamp|`dcount`|
|---|---|
|2018-07-01 00:00:00.0000000|1|
|2018-07-15 00:00:00.0000000|1|

Un usuario se considera activo si cumple los dos criterios siguientes: 
* El usuario se ha detectado en al menos tres días distintos (period = 1D, ActivePeriods = 3).
* El usuario se ha detectado en una ventana de lookback de 8D antes e incluye su apariencia actual.

En la ilustración siguiente, la única apariencia que están activas por este criterio son las siguientes: el usuario A en 7/20 y el usuario B en 7/4 (consulte los resultados del complemento anteriores). La apariencia del usuario B se incluye para la ventana lookback en 7/4, pero no para el intervalo de tiempo de inicio de 6/29-30. 

:::image type="content" source="images/queries/active-users-count.png" alt-text="Ejemplo de recuento de usuarios activos":::

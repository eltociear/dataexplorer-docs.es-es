---
title: active_users_count complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe active_users_count complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f324507d1a4528c5efefc14f7820437383211ca6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519353"
---
# <a name="active_users_count-plugin"></a>plugin active_users_count

Calcula el recuento distinto de valores, donde cada valor ha aparecido en al menos un número mínimo de períodos en un período de búsqueda.

Es útil para calcular recuentos distintos de "fans" solamente, sin incluir apariciones de "no fans". Un usuario se cuenta como un "fan" solo si estuvo activo durante el período de búsqueda. El período de búsqueda solo se utiliza `active` para determinar si un usuario se considera ("fan") o no. La agregación en sí no incluye a los usuarios de la ventana de búsqueda (a diferencia de [sliding_window_counts] (sliding-window-counts-plugin.md) en la que la agregación está sobre la ventana deslizante del período de búsqueda).

```kusto
T | evaluate active_users_count(id, datetime_column, startofday(ago(30d)), startofday(now()), 7d, 1d, 2, 7d, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `,` *Start* `,` `,` *Period* `,` `,` *Bin* `,` *End* `,` `,` *dim2* `,` *IdColumn* `,` *TimelineColumn* *LookbackWindow* *dim1* *ActivePeriodsCount* IdColumn TimelineColumn Start End LookbackWindow Period ActivePeriodsCount Bin [ dim1 dim2 ...] `active_users_count(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColumn*: el nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: el nombre de la columna que representa la línea de tiempo.
* *Inicio*: (opcional) Escalar con el valor del período de inicio del análisis.
* *Final*: (opcional) Escalar con el valor del período final del análisis.
* *LookbackWindow*: Una ventana de tiempo deslizante que define un período en el que se comprueba la apariencia del usuario. El período de retroceso comienza en ([apariencia actual] - [ventana de retroceso]) y termina en ([apariencia actual]). 
* *Período*: Intervalo de tiempo constante escalar para contar como apariencia única (un usuario se contará como activo si aparece en al menos ActivePeriodsCount distinto de este intervalo de tiempo.
* *ActivePeriodsCount*: Número mínimo de períodos activos distintos para decidir si el usuario está activo. Los usuarios activos son aquellos que aparecieron en al menos (igual o mayor que) los períodos activos cuentan.
* *Bin*: Valor constante escalar del período de paso del análisis. Puede ser un valor numérico/datetime/timestamp, o una `week` / `month` / `year`cadena que sea uno de , en cuyo caso todos los períodos serán [el inicio del](startofweekfunction.md)/[mes](startofmonthfunction.md)/de la semana en[consecuencia.](startofyearfunction.md)
* *dim1*, *dim2*, ...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de métricas de actividad.

**Devuelve**

Devuelve una tabla que tiene los valores de recuento distintos para los identificadores que han aparecido en más de ActivePeriodCounts en el período de búsqueda, para cada período de escala de tiempo y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|dim1|..|dim_n|dcount_values|
|---|---|---|---|---|
|tipo: a partir de *TimelineColumn*|..|..|..|long|


**Ejemplos**

Calcular la cantidad semanal de usuarios distintos que aparecieron en al menos en 3 días diferentes durante un período de 8 días anteriores. Período de análisis: Julio 2018.

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

|Timestamp|dcount|
|---|---|
|2018-07-01 00:00:00.0000000|1|
|2018-07-15 00:00:00.0000000|1|

Un usuario se considera activo si se ha visto en al menos 3 días distintos (Período 1d, ActivePeriods-3) en una ventana de búsqueda de 8d anterior a la apariencia actual (incluida la apariencia actual). En la ilustración siguiente, las únicas apariencias que están activas de acuerdo con este criterio, son el Usuario A en el 7/20 y el Usuario B en el 7/4 (ver resultados del plugin arriba). Tenga en cuenta que aunque las apariencias del usuario B en 6/29-30 no están en el intervalo de tiempo de inicio-fin, se incluyen para la ventana de búsqueda del usuario B en el 7/4. 

![texto alternativo](images/queries/active-users-count.png "activos-usuarios-recuento")
---
title: session_count complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe session_count complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 76ce6ca3be1859aba0a94ae155c60342be3f1ad1
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663142"
---
# <a name="session_count-plugin"></a>complemento session_count

Calcula el recuento de sesiones en función de la columna ID a través de una línea de tiempo.

```kusto
T | evaluate session_count(id, datetime_column, startofday(ago(30d)), startofday(now()), 1min, 30min, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `,` *Start* `,` *End* `,` *Bin* `,` `,` `,` `,` *IdColumn* `,` *TimelineColumn* *dim1* *dim2* *LookBackWindow* IdColumn TimelineColumn Start End Bin LookBackWindow [ dim1 dim2 ...] `session_count(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *IdColumn*: el nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: el nombre de la columna que representa la línea de tiempo.
* *Inicio*: Escalar con el valor del período de inicio del análisis.
* *Final*: Escalar con el valor del período final del análisis.
* *Bin*: valor constante escalar del período de paso de análisis de sesión.
* *LookBackWindow*: valor constante escalar que representa el período de búsqueda de sesión. Si el `IdColumn` id de aparece `LookBackWindow` en una ventana de tiempo dentro - la sesión se considera una existente, si no lo hace - la sesión se considera nueva.
* *dim1*, *dim2*, ...: (opcional) lista de las columnas de dimensiones que cortan el cálculo del recuento de sesiones.

**Devuelve**

Devuelve una tabla que tiene los valores de recuento de sesiones para cada período de escala de tiempo y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|dim1|..|dim_n|count_sessions|
|---|---|---|---|---|--|--|--|--|--|--|
|tipo: a partir de *TimelineColumn*|..|..|..|long|


**Ejemplos**


Por el bien del ejemplo, haremos que los datos sean deterministas - una tabla con dos columnas:
- Línea de tiempo: un número de carrera de 1 a 10.000
- Id: Id del usuario de 1 a 50

`Id`aparecen en `Timeline` la ranura específica si `Timeline` se trata de un divisor de (Línea de tiempo % Id á 0).

Esto significa que `Id==1` el evento `Timeline` con aparecerá `Id==2` en `Timeline` cualquier ranura, evento con en cada segundo slot, y así sucesivamente.

Aquí hay algunas 20 líneas de los datos:

```kusto
let _data = range Timeline from 1 to 10000 step 1
| extend __key = 1
| join kind=inner (range Id from 1 to 50 step 1 | extend __key=1) on __key
| where Timeline % Id == 0
| project Timeline, Id;
// Look on few lines of the data
_data
| order by Timeline asc, Id asc
| limit 20
```

|Escala de tiempo|Identificador|
|---|---|
|1|1|
|2|1|
|2|2|
|3|1|
|3|3|
|4|1|
|4|2|
|4|4|
|5|1|
|5|5|
|6|1|
|6|2|
|6|3|
|6|6|
|7|1|
|7|7|
|8|1|
|8|2|
|8|4|
|8|8|

Vamos a definir una sesión en los siguientes términos:`Id`sesión considerada activa siempre y cuando el usuario ( ) aparezca al menos una vez en un período de tiempo de 100 intervalos de tiempo, mientras que la ventana de búsqueda de sesión es 41 intervalos de tiempo.

La siguiente consulta muestra el recuento de sesiones activas según la definición anterior.

```kusto
let _data = range Timeline from 1 to 9999 step 1
| extend __key = 1
| join kind=inner (range Id from 1 to 50 step 1 | extend __key=1) on __key
| where Timeline % Id == 0
| project Timeline, Id;
// End of data definition
_data
| evaluate session_count(Id, Timeline, 1, 10000, 100, 41)
| render linechart 
```

:::image type="content" source="images/queries/example-session-count.png" alt-text="Ejemplo de recuento de sesiones":::

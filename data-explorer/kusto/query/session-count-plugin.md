---
title: 'complemento de session_count: Azure Explorador de datos'
description: En este artículo se describe session_count complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1e173dcba48e8748562bad61e0f16786e957ca83
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550561"
---
# <a name="session_count-plugin"></a>complemento session_count

Calcula el recuento de sesiones basándose en la columna de ID. en una escala de tiempo.

```kusto
T | evaluate session_count(id, datetime_column, startofday(ago(30d)), startofday(now()), 1min, 30min, dim1, dim2, dim3)
```

**Sintaxis**

*T* `| evaluate` `session_count(` *IdColumn* `,` *TimelineColumn* `,` *Start* `,` *End* `,` *bin* `,` *LookBackWindow* [ `,` *DIM1* `,` *dim2* `,` ...]`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *IdColumn*: nombre de la columna con valores de identificador que representan la actividad del usuario. 
* *TimelineColumn*: nombre de la columna que representa la escala de tiempo.
* *Start*: escalar con el valor del período de inicio del análisis.
* *End*: escalar con el valor del período de finalización del análisis.
* *Bin*: valor constante escalar del período de paso de análisis de sesión.
* *LookBackWindow*: valor constante escalar que representa el período de lookback de la sesión. Si el identificador de `IdColumn` aparece en una ventana de tiempo dentro de `LookBackWindow` , se considera que la sesión es una existente. Si no aparece el identificador, se considera que la sesión es nueva.
* *DIM1*, *dim2*,...: (opcional) lista de las columnas de dimensiones que segmentan el cálculo de recuento de sesiones.

**Devuelve**

Devuelve una tabla que tiene los valores de recuento de sesiones para cada período de la escala de tiempo y para cada combinación de dimensiones existente.

El esquema de la tabla de salida es:

|*TimelineColumn*|DIM1|..|dim_n|count_sessions|
|---|---|---|---|---|--|--|--|--|--|--|
|tipo: a partir de *TimelineColumn*|..|..|..|long|


**Ejemplos**

En este ejemplo, los datos son deterministas y usamos una tabla con dos columnas:
- Timeline: un número en ejecución del 1 al 10.000
- ID: ID. del usuario de 1 a 50

`Id`aparece en la `Timeline` ranura específica si es un divisor de `Timeline` (escala de tiempo% ID = = 0).

Un evento con `Id==1` aparecerá en cualquier `Timeline` ranura, un evento con `Id==2` en cada segunda `Timeline` ranura, etc.

A continuación se muestran unas 20 líneas de datos:

<!-- csl: https://help.kusto.windows.net/Samples -->
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

Vamos a definir una sesión en los siguientes términos: la sesión se considera activa mientras el usuario ( `Id` ) aparece al menos una vez en un período de 100 ranuras de tiempo, mientras que la ventana de búsqueda en la sesión es de 41 ranuras.

La siguiente consulta muestra el recuento de sesiones activas según la definición anterior.

<!-- csl: https://help.kusto.windows.net/Samples -->
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

:::image type="content" source="images/session-count-plugin/example-session-count.png" alt-text="Recuento de sesiones de ejemplo" border="false":::

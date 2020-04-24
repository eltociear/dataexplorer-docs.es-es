---
title: sequence_detect complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe sequence_detect complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: afe4b41cc9bf3e81389edfb1fac76472772fa8f6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509187"
---
# <a name="sequence_detect-plugin"></a>sequence_detect plugin

Detecta las apariciones de secuencia en función de los predicados proporcionados.

```kusto
T | evaluate sequence_detect(datetime_column, 10m, 1h, e1 = (Col1 == 'Val'), e2 = (Col2 == 'Val2'), Dim1, Dim2)
```

**Sintaxis**

*T* `| evaluate` `,` `,` `,` `,` `,` *TimelineColumn* `,` *Dim2* *Dim1* *Expr2* *Expr1* *MaxSequenceSpan* *MaxSequenceStepWindow* TimelineColumn`,` MaxSequenceStepWindow MaxSequenceSpan Expr1 Expr2 ..., Dim1 Dim2 ... `sequence_detect` `(``)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *TimelineColumn*: referencia de columna que representa la línea de tiempo, debe estar presente en la expresión de origen
* *MaxSequenceStepWindow*: valor constante escalar del intervalo de tiempo máximo permitido entre 2 pasos secuenciales en la secuencia
* *MaxSequenceSpan*: valor constante escalar del intervalo máximo para que la secuencia complete todos los pasos
* *Expr1*, *Expr2*, ...: expresiones de predicado booleano que definen los pasos de secuencia
* *Dim1*, *Dim2*, ...: expresiones de dimensión que se utilizan para correlacionar secuencias

**Devuelve**

Devuelve una sola tabla donde cada fila de la tabla representa una sola aparición de secuencia:

* *Dim1*, *Dim2*, ...: columnas de dimensión que se utilizaron para correlacionar secuencias.
* *Expr1*_*TimelineColumn*, *Expr2*_*TimelineColumn*, ...: Columnas con valores de tiempo, que representan la línea de tiempo de cada paso de secuencia.
* *Duración*: la ventana de tiempo de secuencia global

**Ejemplos**

### <a name="exploring-storm-events"></a>Explorar eventos de tormenta 

La siguiente consulta examina la tabla StormEvents (estadísticas meteorológicas para 2007) y muestra los casos en los que la secuencia de 'Excessive Heat' fue seguida por 'Wildfire' en un plazo de 5 días.

```kusto
StormEvents
| evaluate sequence_detect(
        StartTime,
        5d,  // step max-time
        5d,  // sequence max-time
        heat=(EventType == "Excessive Heat"), 
        wildfire=(EventType == 'Wildfire'), 
        State)
```

|State|heat_StartTime|wildfire_StartTime|Duration|
|---|---|---|---|
|California|2007-05-08 00:00:00.0000000|2007-05-08 16:02:00.0000000|16:02:00|
|California|2007-05-08 00:00:00.0000000|2007-05-10 11:30:00.0000000|2.11:30:00|
|California|2007-07-04 09:00:00.0000000|2007-07-05 23:01:00.0000000|1.14:01:00|
|DAKOTA DEL SUR|2007-07-23 12:00:00.0000000|2007-07-27 09:00:00.0000000|3.21:00:00|
|TEXAS|2007-08-10 08:00:00.0000000|2007-08-11 13:56:00.0000000|1.05:56:00|
|California|2007-08-31 08:00:00.0000000|2007-09-01 11:28:00.0000000|1.03:28:00|
|California|2007-08-31 08:00:00.0000000|2007-09-02 13:30:00.0000000|2.05:30:00|
|California|2007-09-02 12:00:00.0000000|2007-09-02 13:30:00.0000000|01:30:00|
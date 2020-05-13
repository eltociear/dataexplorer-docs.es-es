---
title: 'complemento de sequence_detect: Azure Explorador de datos'
description: En este artículo se describe sequence_detect complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e8da8a61b285b31f63f346ec82e5ba8a4ac00d27
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372932"
---
# <a name="sequence_detect-plugin"></a>complemento de sequence_detect

Detecta las repeticiones de secuencia basadas en los predicados proporcionados.

```kusto
T | evaluate sequence_detect(datetime_column, 10m, 1h, e1 = (Col1 == 'Val'), e2 = (Col2 == 'Val2'), Dim1, Dim2)
```

**Sintaxis**

*T* `| evaluate` `sequence_detect` `(` *TimelineColumn* `,` *MaxSequenceStepWindow* `,` *MaxSequenceSpan* `,` *expr1* `,` *expr2* `,` ..., *Dim1* `,` *Dim2* `,` ...`)`

**Argumentos**

* *T*: expresión tabular de entrada.
* *TimelineColumn*: la referencia de columna que representa la escala de tiempo debe estar presente en la expresión de origen
* *MaxSequenceStepWindow*: valor constante escalar del intervalo de tiempo máximo permitido entre 2 pasos secuenciales en la secuencia.
* *MaxSequenceSpan*: valor constante escalar del intervalo máximo de la secuencia para completar todos los pasos
* *Expr1*, *expr2*,...: expresiones de predicado booleanas definir pasos de secuencia
* *Dim1*, *Dim2*,...: expresiones de dimensión que se usan para correlacionar secuencias

**Devuelve**

Devuelve una sola tabla en la que cada fila de la tabla representa una única repetición de la secuencia:

* *Dim1*, *Dim2*,...: columnas de dimensión que se usaron para correlacionar secuencias.
* *Expr1*_*TimelineColumn*, *expr2*_*TimelineColumn*,...: columnas con valores de hora que representan la escala de tiempo de cada paso de la secuencia.
* *Duration*: la ventana de tiempo de secuencia general

**Ejemplos**

### <a name="exploring-storm-events"></a>Exploración de eventos de Storm 

La siguiente consulta busca la tabla StormEvents (estadísticas meteorológicas para 2007) y muestra los casos en los que la secuencia de ' exceso de calor ' iba seguida de ' descontrolados ' en 5 días.

<!-- csl: https://help.kusto.windows.net/Samples -->
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

|Estado|heat_StartTime|wildfire_StartTime|Duration|
|---|---|---|---|
|CALIFORNIA|2007-05-08 00:00:00.0000000|2007-05-08 16:02:00.0000000|16:02:00|
|CALIFORNIA|2007-05-08 00:00:00.0000000|2007-05-10 11:30:00.0000000|2.11:30:00|
|CALIFORNIA|2007-07-04 09:00:00.0000000|2007-07-05 23:01:00.0000000|1.14:01:00|
|DAKOTA DEL SUR|2007-07-23 12:00:00.0000000|2007-07-27 09:00:00.0000000|3.21:00:00|
|TEXAS|2007-08-10 08:00:00.0000000|2007-08-11 13:56:00.0000000|1.05:56:00|
|CALIFORNIA|2007-08-31 08:00:00.0000000|2007-09-01 11:28:00.0000000|1,03:28:00|
|CALIFORNIA|2007-08-31 08:00:00.0000000|2007-09-02 13:30:00.0000000|2.05:30:00|
|CALIFORNIA|2007-09-02 12:00:00.0000000|2007-09-02 13:30:00.0000000|01:30:00|

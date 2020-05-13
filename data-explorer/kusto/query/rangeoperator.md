---
title: 'operador de intervalo: Azure Explorador de datos'
description: En este artículo se describe el operador de intervalo en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9dc64e6d91d6832dd57345bf58200848ad5a5db4
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373099"
---
# <a name="range-operator"></a>Operador range

Genera una tabla de valores de una columna única.

Tenga en cuenta que no tiene una entrada de canalización. 

**Sintaxis**

`range`*columnName* `from` *iniciar* `to` *detener* `step` *paso*

**Argumentos**

* *columnName*: el nombre de la columna única de la tabla de salida.
* *Start*: el valor más pequeño de la salida.
* *Stop*: el valor más alto que se está generando en la salida (o un límite en el valor más alto, si el *paso* se recorre por encima de este valor).
* *Step*: la diferencia entre dos valores consecutivos. 

Los argumentos tienen que ser valores numéricos, de fecha o intervalo de tiempo. No pueden tener como referencia las columnas de una tabla. (Si desea calcular el intervalo en función de una tabla de entrada, utilice la función Range, quizás con el operador MV-Expand). 

**Devuelve**

Una tabla con una sola columna denominada *columnName*, cuyos valores son *Start*, *Start* `+` *Step*,... hasta y hasta que se *detenga*.

**Ejemplo**  

Una tabla de medianoche en los últimos siete días. La función bin (floor) se reduce cada vez hasta el inicio del día.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
range LastWeek from ago(7d) to now() step 1d
```

|LastWeek|
|---|
|2015-12-05 09:10:04.627|
|2015-12-06 09:10:04.627|
|...|
|2015-12-12 09:10:04.627|


Una tabla con una única columna denominada `Steps` cuyo tipo es `long` y cuyos valores son `1`, `4` y `7`.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
range Steps from 1 to 8 step 3
```

En el ejemplo siguiente se muestra cómo `range` se puede utilizar el operador para crear una tabla de dimensiones pequeña, ad hoc, que se usa para introducir ceros en los que los datos de origen no tienen valores.

```kusto
range TIMESTAMP from ago(4h) to now() step 1m
| join kind=fullouter
  (Traces
      | where TIMESTAMP > ago(4h)
      | summarize Count=count() by bin(TIMESTAMP, 1m)
  ) on TIMESTAMP
| project Count=iff(isnull(Count), 0, Count), TIMESTAMP
| render timechart  
```

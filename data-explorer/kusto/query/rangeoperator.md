---
title: 'operador de rango: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de rango en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f7ec559a23e28568ce7abd8365cdc502ad05a9b0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510615"
---
# <a name="range-operator"></a>Operador range

Genera una tabla de valores de una columna única.

Tenga en cuenta que no tiene una entrada de canalización. 

**Sintaxis**

`range`*columnName* `from` *start* `to` *stop* `step` *step*

**Argumentos**

* *columnName*: el nombre de la sola columna de la tabla de salida.
* *start*: El valor más pequeño de la salida.
* *stop*: El valor más alto que se genera en la salida (o un límite en el valor más alto, si *pasos* de paso sobre este valor).
* *paso*: La diferencia entre dos valores consecutivos. 

Los argumentos tienen que ser valores numéricos, de fecha o intervalo de tiempo. No pueden tener como referencia las columnas de una tabla. (Si desea calcular el rango basado en una tabla de entrada, utilice la función de rango, tal vez con el operador mv-expand.) 

**Devuelve**

Una tabla con una sola columna denominada *columnName*, cuyos valores son *start*, *start* `+` *step*, ... hasta y *hasta*detener .

**Ejemplo**  

Una tabla de medianoche en los últimos siete días. La función bin (floor) se reduce cada vez hasta el inicio del día.

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

```kusto
range Steps from 1 to 8 step 3
```

En el ejemplo `range` siguiente se muestra cómo se puede utilizar el operador para crear una tabla de dimensiones pequeña, ad hoc, que, a continuación, se utiliza para introducir ceros donde los datos de origen no tienen valores.

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
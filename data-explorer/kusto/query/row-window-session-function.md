---
title: row_window_session() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe row_window_session() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 87e89443bc85125e705bc180ea0cdb9e599c9c13
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510258"
---
# <a name="row_window_session"></a>row_window_session()

`row_window_session()`calcula los valores de inicio de sesión de una columna en un conjunto de [filas serializado.](./windowsfunctions.md#serialized-row-set)

**Sintaxis**

`row_window_session``(` *Expr* `,` *MaxDistanceFromFirst* `,` *MaxDistanceBetweenNeighbors* [`,` *Reiniciar*]`)`

* *Expr* es una expresión cuyos valores se agrupan en sesiones.
  Los valores nulos producen valores nulos y el siguiente valor inicia una nueva sesión.
  *Expr* debe ser una expresión `datetime`escalar de tipo .

* *MaxDistanceFromFirst* establece un criterio para iniciar una nueva sesión: la distancia máxima entre el valor actual de *Expr* y el valor de *Expr* al principio de la sesión.
  Es una constante escalar de `timespan`tipo .

* *MaxDistanceBetweenNeighbors* establece un segundo criterio para iniciar una nueva sesión: la distancia máxima de un valor de *Expr* al siguiente.
  Es una constante escalar de `timespan`tipo .

* *Reiniciar* es una expresión escalar `boolean`escalar opcional de tipo . Si se especifica, cada valor `true` que se evalúa para reiniciar inmediatamente la sesión.

**Devuelve**

La función devuelve los valores al principio de cada sesión.

**Notas**

La función tiene el siguiente modelo de cálculo conceptual:

1. Repase la secuencia de entrada de los valores *Expr* en orden.

2. Para cada valor, determine si establece una nueva sesión.

3. Si establece una nueva sesión, emita el valor de *Expr*. De lo contrario, emita el valor anterior de *Expr*.

La condición que determina si el valor representa una nueva sesión es un OR lógico de lo siguiente:

* Si no había ningún valor de sesión anterior, o el valor de la sesión anterior era null.

* Si el valor de *Expr* es igual o superior al valor de sesión anterior más *MaxDistanceFromFirst*.

* Si el valor de *Expr* es igual o superior al valor anterior de *Expr* más *MaxDistanceBetweenNeighbors*.

**Ejemplos**

En el ejemplo siguiente se muestra cómo calcular los valores `ID` de inicio de sesión `Timestamp` para una tabla con dos columnas, una columna que identifica una secuencia y una columna que proporciona la hora en la que se produjo cada registro. En este ejemplo, una sesión no puede superar 1 hora y continúa siempre que los registros estén separados por menos de 5 minutos.

```kusto
datatable (ID:string, Timestamp:datetime) [
    // ...
]
| sort by ID asc, Timestamp asc
| extend SessionStarted = row_window_session(Timestamp, 1h, 5m, ID != prev(ID))
```
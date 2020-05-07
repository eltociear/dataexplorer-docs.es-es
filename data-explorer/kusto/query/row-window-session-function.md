---
title: 'row_window_session (): Explorador de datos de Azure'
description: En este artículo se describe row_window_session () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 51bc5e26cdc2d002b29ec435a68421ba8768a7a4
ms.sourcegitcommit: 9fe6ee7db15a5cc92150d3eac0ee175f538953d2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2020
ms.locfileid: "82907192"
---
# <a name="row_window_session"></a>row_window_session()

`row_window_session()`calcula los valores de inicio de sesión de una columna en un [conjunto de filas serializado](./windowsfunctions.md#serialized-row-set).

**Sintaxis**

`row_window_session` `(` *`Expr`* `,` *`MaxDistanceFromFirst`* `,` *`MaxDistanceBetweenNeighbors`* [`,` *`Restart`*] `)`

* *`Expr`* es una expresión cuyos valores se agrupan en sesiones.
  Los valores NULL generan valores NULL y el siguiente valor inicia una nueva sesión.
  *`Expr`* debe ser una expresión escalar de `datetime`tipo.

* *`MaxDistanceFromFirst`* establece un criterio para iniciar una nueva sesión: la distancia máxima entre el valor actual de *`Expr`* y el valor de *`Expr`* al principio de la sesión.
  Es una constante escalar de tipo `timespan`.

* *`MaxDistanceBetweenNeighbors`* establece un segundo criterio para iniciar una nueva sesión: la distancia máxima de un valor de *`Expr`* a la siguiente.
  Es una constante escalar de tipo `timespan`.

* *Restart* es una expresión escalar `boolean`opcional de tipo. Si se especifica, cada valor que se evalúa `true` como reiniciará inmediatamente la sesión.

**Devuelve**

La función devuelve los valores al principio de cada sesión.

**Notas**

La función tiene el siguiente modelo de cálculo conceptual:

1. Desplazarse por la secuencia de *`Expr`* valores de entrada en orden.

1. Para cada valor, determine si establece una nueva sesión.

1. Si establece una nueva sesión, emita el valor de *`Expr`*. De lo contrario, emita el valor *`Expr`* anterior de.

La condición que determina si el valor representa una nueva sesión es una o una de las siguientes condiciones:

* Si no hay ningún valor de sesión anterior o el valor de la sesión anterior era null.

* Si el valor de *`Expr`* es igual o mayor que el valor de sesión *`MaxDistanceFromFirst`* anterior más.

* Si el valor de *`Expr`* es igual o mayor que el valor anterior *`Expr`* de *`MaxDistanceBetweenNeighbors`* más.

**Ejemplos**

En el ejemplo siguiente se muestra cómo calcular los valores de inicio de sesión de una tabla con dos `ID` columnas: una columna que identifica una secuencia `Timestamp` y una columna que proporciona la hora a la que se produjo cada registro. En este ejemplo, una sesión no puede superar 1 hora y continúa siempre que los registros tengan menos de 5 minutos de diferencia.

```kusto
datatable (ID:string, Timestamp:datetime) [
    // ...
]
| sort by ID asc, Timestamp asc
| extend SessionStarted = row_window_session(Timestamp, 1h, 5m, ID != prev(ID))
```
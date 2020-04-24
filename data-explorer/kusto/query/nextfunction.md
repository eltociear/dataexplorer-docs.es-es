---
title: next() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe next() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f45e88942fdf9eb23451e1391866f57ca5f0e21a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512128"
---
# <a name="next"></a>next()

Devuelve el valor de una columna de una fila que en algún desplazamiento después de la fila actual en un conjunto de [filas serializada.](./windowsfunctions.md#serialized-row-set)

**Sintaxis**

`next(column)`

`next(column, offset)`

`next(column, offset, default_value)`

**Argumentos**

* `column`: la columna de la que se obtienen los valores.

* `offset`: el desplazamiento para seguir adelante en filas. Cuando no se especifica ningún desplazamiento, se utiliza un desplazamiento predeterminado 1.

* `default_value`: el valor predeterminado que se utilizará cuando no haya filas siguientes de las que tomar el valor. Cuando no se especifica ningún valor predeterminado, se utiliza null.


**Ejemplos**
```kusto
Table | serialize | extend nextA = next(A,1)
| extend diff = A - nextA
| where diff > 1

Table | serialize nextA = next(A,1,10)
| extend diff = A - nextA
| where diff <= 10
```
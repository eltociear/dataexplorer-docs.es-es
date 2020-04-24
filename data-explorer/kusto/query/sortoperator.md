---
title: 'operador de ordenación: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de ordenación en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 638783b28cddc51d64a80096d7d4d6e0f669d354
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507487"
---
# <a name="sort-operator"></a>Operador sort 

Ordena las filas de la tabla de entrada en una o más columnas.

```kusto
T | sort by strlen(country) asc, price desc
```

**Alias**

`order`

**Sintaxis**

*Expresión* `| sort by` T`asc` | `desc`[`nulls first` | `nulls last`]`,` [ ] [ ...] *expression*

**Argumentos**

* *T*: La entrada de tabla que se debe ordenar.
* *expresión*: Una expresión escalar por la que ordenar. El tipo de los valores tiene que ser numérico, fecha, hora o cadena.
* `asc` : ordena en orden ascendente, de bajo a alto. El valor predeterminado es `desc`, descendente de alto a bajo.
* `nulls first`(el valor `asc` predeterminado para order) colocará los `nulls last` valores nulos al principio y (el valor predeterminado para `desc` order) colocará los valores nulos al final.

**Ejemplo**

```kusto
Traces
| where ActivityId == "479671d99b7b"
| sort by Timestamp asc nulls first
```

Todas las filas de la tabla Traces que tienen un determinado valor de `ActivityId`, ordenadas por su marca de tiempo. Si `Timestamp` column contiene valores nulos, estos aparecerán en las primeras líneas del resultado.

Para excluir los valores nulos del resultado agregue un filtro antes de la llamada para ordenar:

```kusto
Traces
| where ActivityId == "479671d99b7b" and isnotnull(Timestamp)
| sort by Timestamp asc
```
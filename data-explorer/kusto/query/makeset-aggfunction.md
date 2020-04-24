---
title: make_set() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_set() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: db11bd528703323d54ff96b228c0b80bbcb86dd9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512689"
---
# <a name="make_set-aggregation-function"></a>make_set() (función de agregación)

Devuelve una matriz `dynamic` (JSON) del conjunto de valores distintivos que *Expr* toma en el grupo.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_set(` *Expr* `,` [ *MaxSize*]`)`

**Argumentos**

* *Expr*: Expresión para el cálculo de la agregación.
* *MaxSize* es un límite entero opcional en el número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

> [!NOTE]
> Una variante heredada y obsoleta `makeset()` de esta función: tiene un límite predeterminado de *MaxSize* 128.

**Devuelve**

Devuelve una matriz `dynamic` (JSON) del conjunto de valores distintivos que *Expr* toma en el grupo.
El criterio de ordenación de la matriz es indefinido.

> [!TIP]
> Para contar solo valores distintos, utilice [dcount()](dcount-aggfunction.md)

**Ejemplo**

```kusto
PageViewLog 
| summarize countries=make_set(country) by continent
```

![texto alternativo](./images/aggregations/makeset.png "makeset")

**Vea también**

* Utilice [`mv-expand`](./mvexpandoperator.md) el operador para la función opuesta.
* [`make_set_if`](./makesetif-aggfunction.md)operador es `make_set`similar a , excepto que también acepta un predicado.
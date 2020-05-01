---
title: 'make_set () (función de agregación): Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe make_set () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: e6eb481423e31e4dfa1b4e6c738ffb525e9aaef7
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618415"
---
# <a name="make_set-aggregation-function"></a>make_set () (función de agregación)

Devuelve una matriz `dynamic` (JSON) del conjunto de valores distintivos que *Expr* toma en el grupo.

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`summarize``make_set(` *Expr* [`,` *MaxSize*]`)`

**Argumentos**

* *Expr*: expresión para el cálculo de agregaciones.
* *MaxSize* es un límite entero opcional del número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

> [!NOTE]
> Una variante heredada y obsoleta de esta función: `makeset()` tiene un límite predeterminado de *MaxSize* = 128.

**Devuelve**

Devuelve una matriz `dynamic` (JSON) del conjunto de valores distintivos que *Expr* toma en el grupo.
El criterio de ordenación de la matriz no está definido.

> [!TIP]
> Para contar solo valores distintos, use [DCont ()](dcount-aggfunction.md)

**Ejemplo**

```kusto
PageViewLog 
| summarize countries=make_set(country) by continent
```

:::image type="content" source="images/makeset-aggfunction/makeset.png" alt-text="Makeset":::

**Vea también**

* Use [`mv-expand`](./mvexpandoperator.md) el operador para la función opuesta.
* [`make_set_if`](./makesetif-aggfunction.md)el operador es similar `make_set`a, salvo que también acepta un predicado.
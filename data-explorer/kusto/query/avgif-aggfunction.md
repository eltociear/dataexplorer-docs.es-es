---
title: avgif() (función de agregación) - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe avgif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 61352be628b7c5a05085c092d0c022deaa0d9b6e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518265"
---
# <a name="avgif-aggregation-function"></a>avgif() (función de agregación)

Calcula el [promedio](avg-aggfunction.md) de *Expr* en todo el `true`grupo para el que *Predicate* se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `avgif(` *Expr*`, `*Predicate*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. Los `null` registros con valores se omiten y no se incluyen en el cálculo.
* *Predicado*: predicado que si es true, el *Expr* valor calculado se agregará al promedio.

**Devuelve**

El valor medio de *Expr* en todo `true`el grupo donde *Predicate* se evalúa como .
 
**Ejemplos**

```kusto
range x from 1 to 100 step 1
| summarize avgif(x, x%2 == 0)
```

|avgif_x|
|---|
|51|
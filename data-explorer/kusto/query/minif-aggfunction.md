---
title: minif() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe minif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0aad254ec01e83bdb07734e5b309c1450512b446
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512366"
---
# <a name="minif-aggregation-function"></a>minif() (función de agregación)

Devuelve el valor mínimo en *Predicate* todo el `true`grupo para el que Predicate se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

Vea también la función - [min(),](min-aggfunction.md) que devuelve el valor mínimo en todo el grupo sin expresión de predicado.

**Sintaxis**

`summarize``minif(` *Expr*`,`*Predicate*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.
* *Predicado*: predicado que si es true, el *Expr* valor calculado se comprobará para el mínimo.

**Devuelve**

El valor mínimo de *Expr* *Predicate* en todo `true`el grupo para el que Predicate se evalúa como .

**Ejemplos**

```kusto
range x from 1 to 100 step 1
| summarize minif(x, x > 50)
```

|minif_x|
|---|
|51|
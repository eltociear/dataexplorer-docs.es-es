---
title: maxif() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe maxif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9be25615f9da61aec6b4d56543f624fa0c24c1c4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512451"
---
# <a name="maxif-aggregation-function"></a>maxif() (función de agregación)

Devuelve el valor máximo en *Predicate* todo el `true`grupo para el que Predicate se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

Vea también - [función max(),](max-aggfunction.md) que devuelve el valor máximo en todo el grupo sin expresión de predicado.

**Sintaxis**

`summarize``maxif(` *Expr*`,`*Predicate*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 
* *Predicado*: predicado que si es true, el *Expr* valor calculado se comprobará para el máximo.

**Devuelve**

El valor máximo de *Expr* *Predicate* en todo `true`el grupo para el que Predicate se evalúa como .

**Ejemplos**

```kusto
range x from 1 to 100 step 1
| summarize maxif(x, x < 50)
```

|maxif_x|
|---|
|49|
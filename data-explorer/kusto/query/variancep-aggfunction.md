---
title: variancep() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe variancep() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 386244806a6fcb3f321eb1a6b40595dc71b2b413
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504597"
---
# <a name="variancep-aggregation-function"></a>variancep() (función de agregación)

Calcula la varianza de *Expr* en todo el grupo, considerando el grupo como una [población.](https://en.wikipedia.org/wiki/Statistical_population) 

* Fórmula usada: ![texto alternativo](./images/aggregations/variance-population.png "varianza-población")

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `variancep(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

El valor de varianza de *Expr* en todo el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), variancep(x) 
```

|list_x|variance_x|
|---|---|
|[ 1, 2, 3, 4, 5]|2|
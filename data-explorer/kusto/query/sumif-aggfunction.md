---
title: sumif() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe sumif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a7d2c96f73b404e8d9acbe9da9defecd6bf1bbf3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506671"
---
# <a name="sumif-aggregation-function"></a>sumif() (función de agregación)

Devuelve una suma de *Expr* para `true`la que *Predicate* se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

También puede utilizar la función [sum(),](sum-aggfunction.md) que suma filas sin expresión de predicado.

**Sintaxis**

resumir `sumif(` *Expr*`,`*Predicate*`)`

**Argumentos**

* *Expr*: expresión para el cálculo de la agregación. 
* *Predicado*: predicado que, si es true, el valor calculado de *Expr*se agregará a la suma. 

**Devuelve**

El valor sum de *Expr* para `true`el que *Predicate* se evalúa como .

> [!TIP]
> Usar `summarize sumif(expr, filter)` en lugar de `where filter | summarize sum(expr)`

**Ejemplo**

```kusto
let T = datatable(name:string, day_of_birth:long)
[
   "John", 9,
   "Paul", 18,
   "George", 25,
   "Ringo", 7
];
T
| summarize sumif(day_of_birth, strlen(name) > 4)
```

|sumif_day_of_birth|
|----|
|32|
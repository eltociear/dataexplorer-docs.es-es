---
title: make_set_if() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_set_if() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1393e063fb0abb91b38a8b9e1edc0110e78b3638
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512655"
---
# <a name="make_set_if-aggregation-function"></a>make_set_if() (función de agregación)

Devuelve `dynamic` una matriz (JSON) del conjunto de valores distintos que *Expr* `true`toma en el grupo, para el que *Predicate* se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_set_if(` *Expr*,`,` *Predicado* [ *MaxSize*]`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.
* *Predicado*: Predicado que tiene que evaluar `true` para que *Expr* se agregue al resultado.
* *MaxSize* es un límite entero opcional en el número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

**Devuelve**

Devuelve `dynamic` una matriz (JSON) del conjunto de valores distintos que *Expr* `true`toma en el grupo, para el que *Predicate* se evalúa como .
El criterio de ordenación de la matriz es indefinido.

> [!TIP]
> Para contar solo los valores distintos, utilice [dcountif()](dcountif-aggfunction.md)

**Vea también**

[`make_set`](./makeset-aggfunction.md)función, que hace lo mismo, sin expresión de predicado.

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
| summarize make_set_if(name, strlen(name) > 4)
```

|set_name|
|----|
|["George", "Ringo"]|
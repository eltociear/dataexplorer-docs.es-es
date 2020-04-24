---
title: make_list_if() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_list_if() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b34c1dad7be709145c622c97b357734c25292bba
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512740"
---
# <a name="make_list_if-aggregation-function"></a>make_list_if() (función de agregación)

Devuelve `dynamic` una matriz (JSON) de todos los valores de *Expr* `true`en el grupo, para el que *Predicate* se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_list_if(` *Expr*,`,` *Predicado* [ *MaxSize*]`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.
* *Predicado*: Predicado que tiene que evaluarse como `true`, para que *Expr* se agregue al resultado.
* *MaxSize* es un límite entero opcional en el número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

**Devuelve**

Devuelve `dynamic` una matriz (JSON) de todos los valores de *Expr* `true`en el grupo, para el que *Predicate* se evalúa como .
Si la entrada `summarize` del operador no está ordenada, el orden de los elementos de la matriz resultante es indefinido.
Si se ordena `summarize` la entrada al operador, el orden de los elementos de la matriz resultante realiza un seguimiento del de la entrada.

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
| summarize make_list_if(name, strlen(name) > 4)
```

|list_name|
|----|
|["George", "Ringo"]|

**Vea también**

[`make_list`](./makelist-aggfunction.md)función, que hace lo mismo, sin expresión de predicado.
---
title: array_iif() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_iif() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/28/2019
ms.openlocfilehash: f99b9aa8a9d081a7f28cd2e5bb8750b15f2fcdac
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663913"
---
# <a name="array_iif"></a>array_iif()

Función iif de elementos en matrices dinámicas.

Otro alias: array_iff().

**Sintaxis**

`array_iif(`*ConditionArray*, *IfTrue*, *IfFalse*]`)`

**Argumentos**

* *conditionArray*: Matriz de entrada de valores *booleanos* o numéricos, debe ser matriz dinámica.
* *ifTrue*: Matriz de entrada de valores o valores primitivos - los valores de resultado cuando el valor correspondiente de *ConditionArray* es *true*.
* *ifFalse*: Matriz de entrada de valores o valores primitivos - los valores de resultado cuando el valor correspondiente de *ConditionArray* es *false*.

**Notas**

* La longitud del resultado es la longitud de *conditionArray*.
* El valor de la condición numérica se trata como *condición* !- *0*.
* El valor de condición no numérico/nulo tendrá null en el índice correspondiente del resultado.
* Los valores que faltan (en matrices de longitud más corta) se tratan como null.

**Devuelve**

Matriz dinámica de los valores tomados de los valores *IfTrue* o *IfFalse* [array], según el valor correspondiente de la matriz Condition.

**Ejemplo**

```kusto
print condition=dynamic([true,false,true]), l=dynamic([1,2,3]), r=dynamic([4,5,6]) 
| extend res=array_iif(condition, l, r)
```

|condición|l|r|res|
|---|---|---|---|
|[verdadero, falso, verdadero]|[1, 2, 3]|[4, 5, 6]|[1, 5, 3]|

---
title: zip() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe zip() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bd407ce652d41471be5b30a15c2c09b9f608edb1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504240"
---
# <a name="zip"></a>zip()

La `zip` función acepta `dynamic` cualquier número de matrices y devuelve una matriz cuyos elementos son cada una matriz que contiene los elementos de las matrices de entrada del mismo índice.

**Sintaxis**

`zip(`*array1* `,` *array2*`, ... )`

**Argumentos**

Entre 2 y 16 matrices dinámicas.

**Ejemplos**

El siguiente ejemplo devuelve `[[1,2],[3,4],[5,6]]`:

```kusto
print zip(dynamic([1,3,5]), dynamic([2,4,6]))
```

El siguiente ejemplo devuelve `[["A",{}], [1,"B"], [1.5, null]]`:

```kusto
print zip(dynamic(["A", 1, 1.5]), dynamic([{}, "B"]))
```

El siguiente ejemplo devuelve `[[1,"one"],[2,"two"],[3,"three"]]`:

```kusto
datatable(a:int, b:string) [1,"one",2,"two",3,"three"]
| summarize a = make_list(a), b = make_list(b)
| project zip(a, b)
```
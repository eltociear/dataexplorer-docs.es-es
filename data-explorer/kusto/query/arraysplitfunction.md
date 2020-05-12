---
title: 'array_split (): Explorador de datos de Azure'
description: En este artículo se describe array_split () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/28/2018
ms.openlocfilehash: 102077c9c1116bd9476c6dae59d993a6379b69bd
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225566"
---
# <a name="array_split"></a>array_split()

Divide una matriz en varias matrices según los índices de división y empaqueta la matriz generada en una matriz dinámica.

**Sintaxis**

`array_split`(*`arr`*, *`indices`*)

**Argumentos**

* *`arr`*: La matriz de entrada que se va a dividir debe ser dinámica.
* *`indices`*: Entero o matriz dinámica de enteros con los índices de división (basados en cero). los valores negativos se convierten en array_length + valor.

**Devuelve**

Matriz dinámica que contiene N + 1 matrices con los valores del intervalo `[0..i1), [i1..i2), ... [iN..array_length)` de `arr` , donde N es el número de índices de entrada y `i1...iN` son los índices.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend arr_split=array_split(arr, 2)
```

|`arr`|`arr_split`|
|---|---|
|[1, 2, 3, 4, 5]|[[1,2], [3, 4, 5]]|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend arr_split=array_split(arr, dynamic([1,3]))
```

|`arr`|`arr_split`|
|---|---|
|[1, 2, 3, 4, 5]|[[1], [2, 3], [4, 5]]|

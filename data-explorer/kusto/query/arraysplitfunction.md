---
title: array_split() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_split() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/28/2018
ms.openlocfilehash: 4d5d8ce5e918f335f1f26f4e3fc045ab0fa6e3a1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518571"
---
# <a name="array_split"></a>array_split()

Divide una matriz en varias matrices según los índices divididos y empaqueta la matriz generada en una matriz dinámica.

**Sintaxis**

`array_split(`*arr*, *índices*`)`

**Argumentos**

* *arr*: Matriz de entrada para dividir, debe ser matriz dinámica.
* *indices*: Matriz entera o dinámica de enteros con los índices divididos (basados en cero), los valores negativos se convierten en array_length + valor.

**Devuelve**

Matriz dinámica que contiene matrices N+1 con los valores en el rango [0..i1), [i1.. i2), ... [iN.. array_length) de arr, donde N es el número de índices de entrada e i1.. iN son los índices.

**Ejemplos**

1.
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend arr_split=array_split(arr, 2)
```
|Arr|arr_split|
|---|---|
|[1,2,3,4,5]|[[1,2],[3,4,5]]|



2.
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend arr_split=array_split(arr, dynamic([1,3]))
```
|Arr|arr_split|
|---|---|
|[1,2,3,4,5]|[[1],[2,3],[4,5]]|
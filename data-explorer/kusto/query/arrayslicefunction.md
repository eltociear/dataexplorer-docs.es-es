---
title: array_slice() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_slice() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/03/2018
ms.openlocfilehash: ed5c320ef639f7545e19bcaabd9b53f4424c959d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518588"
---
# <a name="array_slice"></a>array_slice()

Extrae un segmento de una matriz dinámica.

**Sintaxis**

`array_slice(`*arr*, *start*, *end*]`)`

**Argumentos**

* *arr*: Matriz de entrada de la que se va a extraer el sector, debe ser una matriz dinámica.
* *start*: índice de inicio de base cero (incluido) del sector, los valores negativos se convierten en array_length+start.
* *end*: índice final de base cero (incluido) del sector, los valores negativos se convierten en array_length+end.

Nota: los índices fuera de los límites se ignoran.

**Devuelve**

Matriz dinámica de los valores del rango [start.. fin] del arr.

**Ejemplos**

1.
```kusto
print arr=dynamic([1,2,3]) 
| extend sliced=array_slice(arr, 1, 2)
```
|Arr|Rebanado|
|---|---|
|[1,2,3]|[2,3]|


2.
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend sliced=array_slice(arr, 2, -1)
```
|Arr|Rebanado|
|---|---|
|[1,2,3,4,5]|[3,4,5]|


3.
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend sliced=array_slice(arr, -3, -2)
```
|Arr|Rebanado|
|---|---|
|[1,2,3,4,5]|[3,4]|
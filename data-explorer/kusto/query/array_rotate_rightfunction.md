---
title: array_rotate_right() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_rotate_right() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 4f45db14f6ca2fe990f8d139c608ebed1915aa16
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518809"
---
# <a name="array_rotate_right"></a>array_rotate_right()

`array_rotate_right()`gira los valores dentro de una matriz a la derecha.

**Sintaxis**

`array_rotate_right(`*arr*, *rotate_count*`)`

**Argumentos**

* *arr*: Matriz de entrada para dividir, debe ser matriz dinámica.
* *rotate_count*: Entero que especifica el número de posiciones en las que los elementos de la matriz se rotarán a la derecha. Si el valor es negativo, los elementos se rotarán a la izquierda.

**Devuelve**

Matriz dinámica que contiene la misma cantidad de elementos que en la matriz original, donde cada elemento se giró según *rotate_count*.

**Vea también**

* Para rotar la matriz a la izquierda, consulte [array_rotate_left()](array_rotate_leftfunction.md).
* Para desplazar la matriz a la izquierda, consulte [array_shift_left()](array_shift_leftfunction.md).
* Para desplazar la matriz a la derecha, consulte [array_shift_right()](array_shift_rightfunction.md).

**Ejemplos**

* Girando a la derecha por dos posiciones:

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_rotated=array_rotate_right(arr, 2)
    ```
    
    |Arr|arr_rotated|
    |---|---|
    |[1,2,3,4,5]|[4,5,1,2,3]|

* Girar a la izquierda por dos posiciones mediante el uso de valor rotate_count negativo:

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_rotated=array_rotate_right(arr, -2)
    ```
    
    |Arr|arr_rotated|
    |---|---|
    |[1,2,3,4,5]|[3,4,5,1,2]|
---
title: array_shift_right() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe array_shift_right() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 73f87d4c5ce1a841404e438e0e5647089b38785f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518775"
---
# <a name="array_shift_right"></a>array_shift_right()

`array_shift_right()`desplaza los valores dentro de una matriz a la derecha.

**Sintaxis**

`array_shift_right(`*arr*, *shift_count* [, *fill_value* ]`)`

**Argumentos**

* *arr*: Matriz de entrada para dividir, debe ser matriz dinámica.
* *shift_count*: Entero que especifica el número de posiciones en las que los elementos de la matriz se desplazarán a la derecha. Si el valor es negativo, los elementos se desplazarán a la izquierda.
* *fill_value*: valor escalar que se utiliza para insertar elementos en lugar de los que se han desplazado y eliminado. Predeterminado: valor nulo o cadena vacía (dependiendo del tipo *arr).*

**Devuelve**

Matriz dinámica que contiene la misma cantidad de elementos que en la matriz original, donde cada elemento se desplazaba según *shift_count*. Los nuevos elementos que se agregan en lugar de los elementos que se quitan tendrán el valor de *fill_value*.

**Vea también**

* Para desplazar la matriz a la izquierda, consulte [array_shift_left()](array_shift_leftfunction.md).
* Para girar la matriz a la derecha, consulte [array_rotate_right()](array_rotate_rightfunction.md).
* Para la matriz giratoria a la izquierda, consulte [array_rotate_left()](array_rotate_leftfunction.md).

**Ejemplos**

* Cambio a la derecha por dos posiciones:

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_right(arr, 2)
    ```
    
    |Arr|arr_shift|
    |---|---|
    |[1,2,3,4,5]|[null,null,1,2,3]|

* Cambiar a la derecha por dos posiciones y añadir un valor predeterminado:

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_right(arr, 2, -1)
    ```
    
    |Arr|arr_shift|
    |---|---|
    |[1,2,3,4,5]|[-1,-1,1,2,3]|


* Desplazamiento a la izquierda por dos posiciones mediante un valor shift_count negativo:

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_right(arr, -2, -1)
    ```
    
    |Arr|arr_shift|
    |---|---|
    |[1,2,3,4,5]|[3,4,5,-1,-1]|
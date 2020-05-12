---
title: 'array_rotate_right (): Explorador de datos de Azure'
description: En este artículo se describe array_rotate_right () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 23f16885d1988823fe2b301035c6ba5d54471136
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225775"
---
# <a name="array_rotate_right"></a>array_rotate_right()

`array_rotate_right()`gira los valores dentro de una matriz a la derecha.

**Sintaxis**

`array_rotate_right(`*ARR*, *rotate_count*`)`

**Argumentos**

* *ARR*: matriz de entrada que se va a dividir; debe ser una matriz dinámica.
* *rotate_count*: entero que especifica el número de posiciones que los elementos de la matriz se girarán a la derecha. Si el valor es negativo, los elementos se girarán a la izquierda.

**Devuelve**

Matriz dinámica que contiene la misma cantidad de elementos que en la matriz original, donde cada elemento se giró según *rotate_count*.

**Vea también**

* Para rotar la matriz a la izquierda, consulte [array_rotate_left ()](array_rotate_leftfunction.md).
* Para desplazarse por la matriz a la izquierda, consulte [array_shift_left ()](array_shift_leftfunction.md).
* Para desplazarse por la matriz a la derecha, vea [array_shift_right ()](array_shift_rightfunction.md).

**Ejemplos**

* Girar a la derecha dos posiciones:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_rotated=array_rotate_right(arr, 2)
    ```
    
    |ARR|arr_rotated|
    |---|---|
    |[1, 2, 3, 4, 5]|[4, 5, 1, 2, 3]|

* Girar a la izquierda dos posiciones mediante el valor de rotate_count negativo:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_rotated=array_rotate_right(arr, -2)
    ```
    
    |ARR|arr_rotated|
    |---|---|
    |[1, 2, 3, 4, 5]|[3, 4, 5, 1, 2]|

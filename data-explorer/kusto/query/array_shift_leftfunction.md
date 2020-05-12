---
title: 'array_shift_left (): Explorador de datos de Azure'
description: En este artículo se describe array_shift_left () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: f901775dc8bb26c6fb863eefa9b221a89ecf5d1b
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225707"
---
# <a name="array_shift_left"></a>array_shift_left()

`array_shift_left()`desplaza los valores dentro de una matriz a la izquierda.

**Sintaxis**

`array_shift_left(`*`arr`*, *`shift_count`* `[,` *`fill_value`* ]`)`

**Argumentos**

* *`arr`*: La matriz de entrada que se va a dividir debe ser dinámica.
* *`shift_count`*: Entero que especifica el número de posiciones que los elementos de la matriz se desplazarán hacia la izquierda. Si el valor es negativo, los elementos se desplazarán a la derecha.
* *`fill_value`*: Valor escalar que se usa para insertar elementos en lugar de los que se desplazaron y quitaron. Valor predeterminado: null o una cadena vacía (dependiendo del *`arr`* tipo).

**Devuelve**

Matriz dinámica que contiene el mismo número de elementos que en la matriz original. Cada elemento se ha desplazado según *shift_count*. Los nuevos elementos que se agregan en lugar de los elementos quitados tendrán un valor de *fill_value*.

**Vea también**

* Para desplazarse a la derecha de la matriz, vea [array_shift_right ()](array_shift_rightfunction.md).
* Para rotar la matriz a la derecha, consulte [array_rotate_right ()](array_rotate_rightfunction.md).
* Para rotar la matriz a la izquierda, consulte [array_rotate_left ()](array_rotate_leftfunction.md).

**Ejemplos**

* Desplazarse a la izquierda en dos posiciones:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_left(arr, 2)
    ```
    
    |`arr`|`arr_shift`|
    |---|---|
    |[1, 2, 3, 4, 5]|[3, 4, 5, null, null]|

* Desplazar a la izquierda dos posiciones y agregar el valor predeterminado:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_left(arr, 2, -1)
    ```
    
    |`arr`|`arr_shift`|
    |---|---|
    |[1, 2, 3, 4, 5]|[3, 4, 5,-1,-1]|


* Desplazarse a la derecha dos posiciones mediante el valor de *shift_count* negativo:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_left(arr, -2, -1)
    ```
    
    |`arr`|`arr_shift`|
    |---|---|
    |[1, 2, 3, 4, 5]|[-1,-1, 1, 2, 3]|

---
title: 'array_shift_right (): Explorador de datos de Azure'
description: En este artículo se describe array_shift_right () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 28a44365d6d79bf30ec188146d989f2af2ad12c1
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550663"
---
# <a name="array_shift_right"></a>array_shift_right()

`array_shift_right()`desplaza los valores dentro de una matriz a la derecha.

**Sintaxis**

`array_shift_right(`*`arr`*, *`shift_count`* [, *`fill_value`* ]`)`

**Argumentos**

* *`arr`*: La matriz de entrada que se va a dividir debe ser dinámica.
* *`shift_count`*: Entero que especifica el número de posiciones que los elementos de la matriz se desplazarán hacia la derecha. Si el valor es negativo, los elementos se desplazarán a la izquierda.
* *`fill_value`*: valor escalar que se usa para insertar elementos en lugar de los que se desplazaron y quitaron. Valor predeterminado: null o una cadena vacía (dependiendo del tipo *ARR* ).

**Devuelve**

Matriz dinámica que contiene la misma cantidad de elementos que en la matriz original. Cada elemento se ha desplazado según *`shift_count`* . Los nuevos elementos que se agreguen en lugar de los elementos quitados tendrán un valor de *`fill_value`* .

**Vea también**

* Para desplazar la matriz a la izquierda, consulte [array_shift_left ()](array_shift_leftfunction.md).
* Para rotar la matriz a la derecha, consulte [array_rotate_right ()](array_rotate_rightfunction.md).
* Para rotar la matriz a la izquierda, consulte [array_rotate_left ()](array_rotate_leftfunction.md).

**Ejemplos**

* Desplazarse a la derecha dos posiciones:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_right(arr, 2)
    ```
    
    |ARR|arr_shift|
    |---|---|
    |[1, 2, 3, 4, 5]|[null, null, 1, 2, 3]|

* Desplazarse a la derecha dos posiciones y agregar un valor predeterminado:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_right(arr, 2, -1)
    ```
    
    |ARR|arr_shift|
    |---|---|
    |[1, 2, 3, 4, 5]|[-1,-1, 1, 2, 3]|

* Desplazar a la izquierda dos posiciones mediante un valor de shift_count negativo:

    <!-- csl: https://help.kusto.windows.net:443/Samples -->
    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_right(arr, -2, -1)
    ```
    
    |ARR|arr_shift|
    |---|---|
    |[1, 2, 3, 4, 5]|[3, 4, 5,-1,-1]|
    
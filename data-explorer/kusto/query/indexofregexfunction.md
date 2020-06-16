---
title: 'indexof_regex (): Explorador de datos de Azure'
description: En este artículo se describe indexof_regex () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 72797b54c3ba431b4a846f9e9661e9693359cceb
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780463"
---
# <a name="indexof_regex"></a>indexof_regex()

La función devuelve el índice de base cero de la primera aparición de una cadena especificada dentro de la cadena de entrada. Las coincidencias de cadena simple no se superponen.

Vea [`indexof()`](indexoffunction.md).

**Sintaxis**

`indexof_regex(`*origen* `,` de *búsqueda* `[,` *start_index* `[,` *longitud* `[,` *repetición*`]]])`

**Argumentos**

|Argumentos     | Descripción                                     |Obligatorio u opcional|
|--------------|-------------------------------------------------|--------------------|
|source        | Cadena de entrada                                    |Requerido            |
|buscar        | Cadena que se va a buscar                                  |Requerido            |
|start_index   | Posición de inicio de la búsqueda                           |Opcionales            |
|length        | Número de posiciones de caracteres que se van a examinar. -1 define una longitud ilimitada |Opcionales            |
|occurrence    | Busque el índice de la apariencia N-ésima del patrón. 
                 El valor predeterminado es 1, el índice de la primera aparición. |Opcionales            |

**Devuelve**

Posición de índice de base cero de *lookup*.

* Devuelve-1 si la cadena no se encuentra en la entrada.
* Devuelve *null* si:
     * start_index es menor que 0.
     * la repetición es menor que 0.
     * el parámetro de longitud es menor que-1.


**Ejemplos**

```kusto
print
 idx1 = indexof_regex("abcabc", "a.c") // lookup found in input string
 , idx2 = indexof_regex("abcabcdefg", "a.c", 0, 9, 2)  // lookup found in input string
 , idx3 = indexof_regex("abcabc", "a.c", 1, -1, 2)  // there is no second occurrence in the search range
 , idx4 = indexof_regex("ababaa", "a.a", 0, -1, 2)  // Plain string matches do not overlap so full lookup can't be found
 , idx5 = indexof_regex("abcabc", "a|ab", -1)  // invalid input
```

|idx1|idx2|idx3|idx4|idx5|
|----|----|----|----|----|
|0   |3   |-1  |-1  |    |

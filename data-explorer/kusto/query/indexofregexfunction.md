---
title: indexof_regex() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe indexof_regex() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6da0523e85bab4883c50708ffe3f7d087fdd8c8f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513896"
---
# <a name="indexof_regex"></a>indexof_regex()

Función notifica el índice de base cero de la primera aparición de una cadena especificada dentro de la cadena de entrada. Las coincidencias de cadena sin formato no se superponen. 

Ver [`indexof()`](indexoffunction.md).

**Sintaxis**

`indexof_regex(`*búsqueda*`,`*de*`[,`la fuente*start_index*`[,`la*ocurrencia* *de la longitud*`[,``]]])`

**Argumentos**

* *fuente*: cadena de entrada.  
* *búsqueda*: cadena a buscar.
* *start_index*: posición de inicio de búsqueda (opcional).
* *longitud*: número de posiciones de caracteres a examinar, -1 definir longitud ilimitada (opcional).
* *ocurrencia*: es la ocurrencia Predeterminado 1 (opcional).

**Devuelve**

Posición de *búsqueda*de índice de base cero.

Devuelve -1 si la cadena no se encuentra en la entrada.
En caso de irrelevante (menos de 0) *start_index*, *ocurrencia* o (menos de -1) parámetro de *longitud* - devuelve *null*.


**Ejemplos**
```kusto
print
 idx1 = indexof_regex("abcabc", "a.c") // lookup found in input string
 , idx2 = indexof_regex("abcabcdefg", "a.c", 0, 9, 2)  // lookup found in input string
 , idx3 = indexof_regex("abcabc", "a.c", 1, -1, 2)  // there is no second occurrence in the search range
 , idx4 = indexof_regex("ababaa", "a.a", 0, -1, 2)  // Plain string matches do not overlap so full lookup can't be found
 , idx5 = indexof_regex("abcabc", "a|ab", -1)  // invalid input
```

|idx1|idx2|idx3|idx4|idx5
|----|----|----|----|----
|0   |3   |-1  |-1  |    
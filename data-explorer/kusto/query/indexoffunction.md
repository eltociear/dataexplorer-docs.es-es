---
title: indexof() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe indexof() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 698cc7c13c3d665f9f5cfe25a31269dc763c51fb
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513947"
---
# <a name="indexof"></a>indexof()

Función notifica el índice de base cero de la primera aparición de una cadena especificada dentro de la cadena de entrada.

Si la cadena de búsqueda o de entrada no es de tipo cadena, convierte forzosamente el valor en string.

Ver [`indexof_regex()`](indexofregexfunction.md).

**Sintaxis**

`indexof(`*búsqueda*`,`*de*`[,`la fuente*start_index*`[,`la*ocurrencia* *de la longitud*`[,``]]])`

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
 idx1 = indexof("abcdefg","cde")    // lookup found in input string
 , idx2 = indexof("abcdefg","cde",1,4) // lookup found in researched range 
 , idx3 = indexof("abcdefg","cde",1,2) // search starts from index 1, but stops after 2 chars, so full lookup can't be found
 , idx4 = indexof("abcdefg","cde",3,4) // search starts after occurrence of lookup
 , idx5 = indexof("abcdefg","cde",-1)  // invalid input
 , idx6 = indexof(1234567,5,1,4)       // two first parameters were forcibly casted to strings "12345" and "5"
 , idx7 = indexof("abcdefg","cde",2,-1)  // lookup found in input string
 , idx8 = indexof("abcdefgabcdefg", "cde", 1, 10, 2)   // lookup found in input range
 , idx9 = indexof("abcdefgabcdefg", "cde", 1, -1, 3)   // the third occurrence of lookup is not in researched range
```

|idx1|idx2|idx3|idx4|idx5|idx6|idx7|idx8|idx9|
|----|----|----|----|----|----|----|----|----|
|2   |2   |-1  |-1  |    |4   |2   |9   |-1  |

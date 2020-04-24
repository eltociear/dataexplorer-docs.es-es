---
title: array_index_of() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_index_of() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/22/2020
ms.openlocfilehash: f68c9385c55cedb4491033d137af087dafd26aa0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518622"
---
# <a name="array_index_of"></a>array_index_of()

Busca en la matriz el elemento especificado y devuelve su posición.

**Sintaxis**

`array_index_of(`*array*,*valor*`)`

**Argumentos**

* *array*: Matriz de entrada para buscar.
* *valor*: Valor a buscar. El valor debe ser de tipo long, integer, double, datetime, timespan, decimal, string o guid.

**Devuelve**

Posición de búsqueda de índice de base cero.
Devuelve -1 si el valor no se encuentra en la matriz.

**Ejemplo**

```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=array_index_of(arr, "example")
```

|Resultado|
|---|
|3|

**Vea también**

Si solo desea comprobar si existe un valor en una matriz, pero no está interesado en su posición, puede usar [set_has_element (arr, value)](sethaselementfunction.md) - esto mejorará la legibilidad de la consulta, pero ambas funciones en cuanto al rendimiento son las mismas.
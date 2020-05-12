---
title: 'array_index_of (): Explorador de datos de Azure'
description: En este artículo se describe array_index_of () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/22/2020
ms.openlocfilehash: 99044d8762a1c7c7e86fb2633a8226ef48d66b55
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225639"
---
# <a name="array_index_of"></a>array_index_of()

Busca el elemento especificado en la matriz y devuelve su posición.

**Sintaxis**

`array_index_of(`*matriz*,*valor*`)`

**Argumentos**

* *matriz*: matriz de entrada en la que se va a buscar.
* *valor*: valor que se va a buscar. El valor debe ser de tipo Long, integer, Double, DateTime, TimeSpan, decimal, String o GUID.

**Devuelve**

Posición de índice de base cero de lookup.
Devuelve-1 si el valor no se encuentra en la matriz.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=array_index_of(arr, "example")
```

|Resultado|
|---|
|3|

**Vea también**

Si solo desea comprobar si un valor existe en una matriz, pero no le interesa su posición, puede usar [set_has_element ( `arr` , `value` )](sethaselementfunction.md). Esta función mejorará la legibilidad de la consulta. Ambas funciones tienen el mismo rendimiento.

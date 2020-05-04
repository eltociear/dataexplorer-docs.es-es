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
ms.openlocfilehash: 27b956ee54ef22f55b3a0ceae97fceb41aadf5c3
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737358"
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

```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=array_index_of(arr, "example")
```

|Resultado|
|---|
|3|

**Vea también**

Si solo desea comprobar si un valor existe en una matriz, pero no le interesa su posición, puede usar [set_has_element (`arr`, `value`)](sethaselementfunction.md). Esta función mejorará la legibilidad de la consulta. Ambas funciones tienen el mismo rendimiento.

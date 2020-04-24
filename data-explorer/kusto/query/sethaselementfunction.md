---
title: set_has_element() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe set_has_element() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: 256e01646c6ecd39a8a589299acd6620008ece28
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507776"
---
# <a name="set_has_element"></a>set_has_element()

Determina si el conjunto especificado contiene el elemento especificado.

**Sintaxis**

`set_has_element(`*array*,*valor*`)`

**Argumentos**

* *array*: Matriz de entrada para buscar.
* *valor*: Valor a buscar. El valor debe `long`ser `integer` `double`de `datetime` `timespan`tipo `decimal` `string`, `guid`, , , , , , , o .

**Devuelve**

True o false dependiendo de si el valor existe en la matriz.

**Ejemplo**

```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=set_has_element(arr, "example")
```

|Resultado|
|---|
|1|

**Vea también**

Si también está interesado en la posición en la que existe el valor en la matriz, puede utilizar [array_index_of(arr, value)](arrayindexoffunction.md). Ambas funciones son iguales, en cuanto al rendimiento.
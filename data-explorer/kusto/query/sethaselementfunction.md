---
title: 'set_has_element (): Explorador de datos de Azure'
description: En este artículo se describe set_has_element () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: 8e96b97754600d308526a4cb059907521fda0521
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83862935"
---
# <a name="set_has_element"></a>set_has_element()

Determina si el conjunto especificado contiene el elemento especificado.

**Sintaxis**

`set_has_element(`*matriz*,*valor*`)`

**Argumentos**

* *matriz*: matriz de entrada en la que se va a buscar.
* *valor*: valor que se va a buscar. El valor debe ser de tipo `long` , `integer` , `double` , `datetime` , `timespan` , `decimal` , `string` o `guid` .

**Devuelve**

True o false, dependiendo de si el valor existe en la matriz.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=set_has_element(arr, "example")
```

|Resultado|
|---|
|1|

**Vea también**

Use [`array_index_of(arr, value)`](arrayindexoffunction.md) para buscar la posición en la que el valor existe en la matriz. Ambas funciones tienen el mismo rendimiento.

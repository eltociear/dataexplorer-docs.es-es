---
title: 'pack_array (): Explorador de datos de Azure'
description: En este artículo se describe pack_array () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: f20fa8f07368052691334ab65eec666e9a3d568e
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271356"
---
# <a name="pack_array"></a>pack_array()

Empaqueta todos los valores de entrada en una matriz dinámica.

**Sintaxis**

`pack_array(`*Expr1* `[` , ` *Expr2*]` ) '

**Argumentos**

* *Expr1... N*: expresiones de entrada que se van a empaquetar en una matriz dinámica.

**Devuelve**

Matriz dinámica que incluye los valores de expr1, expr2,..., Exprn.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project pack_array(x,y,z)
```

|Column1|
|---|
|[1, 2, 4]|
|[2, 4, 8]|
|[3, 6, 12]|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = tostring(x * 2)
| extend z = (x * 2) * 1s
| project pack_array(x,y,z)
```

|Column1|
|---|
|[1, "2", "00:00:02"]|
|[2, "4", "00:00:04"]|
|[3, "6", "00:00:06"]|

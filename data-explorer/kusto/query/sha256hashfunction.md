---
title: 'hash_sha256 (): Explorador de datos de Azure'
description: En este artículo se describe hash_sha256 () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 32fa2f3ffefdbf1f14ed87e8e89444de322408c3
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372313"
---
# <a name="hash_sha256"></a>hash_sha256()

Devuelve un valor hash SHA256 para el valor de entrada.

**Sintaxis**

`hash_sha256(`*fuentes*`)`

**Argumentos**

* *origen*: el valor al que se va a aplicar un algoritmo hash.

**Devuelve**

Valor hash SHA256 del valor escalar dado, codificado como una cadena hexadecimal (una cadena de caracteres, cada dos de los cuales representa un único número hexadecimal entre 0 y 255).

> [!WARNING]
> Se garantiza que el algoritmo utilizado por esta función (SHA256) no se modificará en el futuro, pero es muy complejo de calcular. En su lugar, se recomienda a los usuarios que necesiten una función hash "ligera" para la duración de una sola consulta usar la función [hash ()](./hashfunction.md) .

**Ejemplos**

```kusto
hash_sha256("World")                   // 78ae647dc5544d227130a0682a51e30bc7777fbb6d8a8f17007463a3ecd1d524
hash_sha256(datetime("2015-01-01"))    // e7ef5635e188f5a36fafd3557d382bbd00f699bd22c671c3dea6d071eb59fbf8
```

En el ejemplo siguiente se usa la función hash_sha256 para ejecutar una consulta en la columna StartTime de los datos.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where hash_sha256(StartTime) == 0
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State 
| top 5 by StormCount desc
```

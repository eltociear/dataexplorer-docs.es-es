---
title: hash_sha256() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe hash_sha256() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2147f4e9f2bd3d7df8f75ac704a4e4808e69bb3c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507606"
---
# <a name="hash_sha256"></a>hash_sha256()

Devuelve un valor hash sha256 para el valor de entrada.

**Sintaxis**

`hash_sha256(`*Fuente*`)`

**Argumentos**

* *source*: El valor que se va a aplicar un algoritmo hash.

**Devuelve**

El valor hash sha256 del escalar dado, codificado como una cadena hexadecimal (una cadena de caracteres, cada uno de los cuales representan un único número hexadecimal entre 0 y 255).

> [!WARNING]
> Se garantiza que el algoritmo utilizado por esta función (SHA256) no se modifique en el futuro, pero es muy complejo de calcular. Los usuarios que necesitan una función hash "ligera" durante una sola consulta se recomienda utilizar la función [hash()](./hashfunction.md) en su lugar.

**Ejemplos**

```kusto
hash_sha256("World")                   // 78ae647dc5544d227130a0682a51e30bc7777fbb6d8a8f17007463a3ecd1d524
hash_sha256(datetime("2015-01-01"))    // e7ef5635e188f5a36fafd3557d382bbd00f699bd22c671c3dea6d071eb59fbf8
```

En el ejemplo siguiente se utiliza la función hash_sha256 para ejecutar una consulta en la columna StartTime de los datos

```kusto
StormEvents 
| where hash_sha256(StartTime) == 0
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State 
| top 5 by StormCount desc
```
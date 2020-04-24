---
title: iif() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe iif() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 36ba98b9677055dffce32911d80e67a9161b673b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514049"
---
# <a name="iif"></a>iif()

Evalúa el primer argumento (el predicado) y devuelve el valor del segundo o tercer `true` argumento, `false` dependiendo de si el predicado se evaluó a (segundo) o (tercero).

El segundo y tercer argumento deben ser del mismo tipo.

**Sintaxis**

`iif(`*predicado* `,` *ifTrue* `,` *ifFalse*`)`

**Argumentos**

* *predicado:* expresión que `boolean` se evalúa como un valor.
* *ifTrue*: Una expresión que se evalúa y su valor devuelto `true`por la función si *el predicado* se evalúa como .
* *ifFalse*: Una expresión que se evalúa y su valor devuelto `false`por la función si *el predicado* se evalúa como .

**Devuelve**

Esta función devuelve el valor de *ifTrue* si *predicate* se evalúa como `true`, o el valor de *ifFalse* en caso contrario.

**Ejemplo**

```kusto
T 
| extend day = iif(floor(Timestamp, 1d)==floor(now(), 1d), "today", "anotherday")
```

Un alias [`iff()`](ifffunction.md)para .
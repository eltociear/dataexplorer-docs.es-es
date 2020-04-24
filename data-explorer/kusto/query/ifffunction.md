---
title: iff() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe iff() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d5eb8af5fd98fb24c677390c314d112e5634cacf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514066"
---
# <a name="iff"></a>iff()

Evalúa el primer argumento (el predicado) y devuelve el valor del segundo o tercer `true` argumento, `false` dependiendo de si el predicado se evaluó a (segundo) o (tercero).

El segundo y tercer argumento deben ser del mismo tipo.

**Sintaxis**

`iff(`*predicado* `,` *ifTrue* `,` *ifFalse*`)`

**Argumentos**

* *predicado:* expresión que `boolean` se evalúa como un valor.
* *ifTrue*: Una expresión que se evalúa y su valor devuelto `true`por la función si *el predicado* se evalúa como .
* *ifFalse*: Una expresión que se evalúa y su valor devuelto `false`por la función si *el predicado* se evalúa como .

**Devuelve**

Esta función devuelve el valor de *ifTrue* si *predicate* se evalúa como `true`, o el valor de *ifFalse* en caso contrario.

**Ejemplo**

```kusto
T 
| extend day = iff(floor(Timestamp, 1d)==floor(now(), 1d), "today", "anotherday")
```
---
title: binary_or() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe binary_or() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1f6d68137ded3b164ca25d82e0c17b6fef6fc1a1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517602"
---
# <a name="binary_or"></a>binary_or()

Devuelve un resultado de `or` la operación bit a bit de los dos valores. 

```kusto
binary_or(x,y)
```

**Sintaxis**

`binary_or(`*num1* `,` *num2*`)`

**Argumentos**

* *num1*, *num2*: números largos.

**Devuelve**

Devuelve la operación OR lógica en un par de números: num1 num2.
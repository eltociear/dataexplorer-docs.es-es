---
title: binary_xor() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe binary_xor() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c2f487aa44f8885cbb443c31b8bb3a503e1a76fa
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517534"
---
# <a name="binary_xor"></a>binary_xor()

Devuelve un resultado de `xor` la operación bit a bit de los dos valores.

```kusto
binary_xor(x,y)
```

**Sintaxis**

`binary_xor(`*num1* `,` *num2*`)`

**Argumentos**

* *num1*, *num2*: números largos.

**Devuelve**

Devuelve la operación XOR lógica en un par de números: num1 - num2.
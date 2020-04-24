---
title: binary_and() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe binary_and() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 8c5501218c1ddb69a73f685fda78f3b3482afb5c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517687"
---
# <a name="binary_and"></a>binary_and()

Devuelve un resultado de `and` la operación bit a bit entre dos valores.

```kusto
binary_and(x,y) 
```

**Sintaxis**

`binary_and(`*num1* `,` *num2*`)`

**Argumentos**

* *num1*, *num2*: números largos.

**Devuelve**

Devuelve la operación AND lógica en un par de números: num1 & num2.
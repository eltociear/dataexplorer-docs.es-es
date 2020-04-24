---
title: binary_shift_left() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe binary_shift_left() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 15e2bee789e627709ccfedde8eccead7f2578b51
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517568"
---
# <a name="binary_shift_left"></a>binary_shift_left()

Devuelve la operación de desplazamiento binario a la izquierda en un par de números.

```kusto
binary_shift_left(x,y)  
```

**Sintaxis**

`binary_shift_left(`*num1* `,` *num2*`)`

**Argumentos**

* *num1*, *num2*: números int.

**Devuelve**

Devuelve la operación de desplazamiento binario a la izquierda en un par de números: num1 << (num2%64).
Si n es negativo, se devuelve un valor NULL.
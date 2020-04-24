---
title: binary_shift_right() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe binary_shift_right() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a94c8c695f1c5d16ee0a7e3a92882486b8a8ef5d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517551"
---
# <a name="binary_shift_right"></a>binary_shift_right()

Devuelve la operación de desplazamiento binario a la derecha en un par de números.

```kusto
binary_shift_right(x,y) 
```

**Sintaxis**

`binary_shift_right(`*num1* `,` *num2*`)`

**Argumentos**

* *num1*, *num2*: números largos.

**Devuelve**

Devuelve la operación de desplazamiento binario a la derecha en un par de números: num1 >> (num2%64).
Si n es negativo, se devuelve un valor NULL.
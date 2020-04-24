---
title: rand() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe rand() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d3638d4b979b7318f58efec0bed0da4c31896a9d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510649"
---
# <a name="rand"></a>rand()

Devuelve un número aleatorio.

```kusto
rand()
rand(1000)
```

**Sintaxis**

* `rand()`- devuelve un `real` valor de tipo con una distribución uniforme en el intervalo [0.0, 1.0).
* `rand(`*N* `)` - devuelve un `real` valor de tipo elegido con una distribución uniforme del conjunto de valores de 0,0, 1,0, ..., *N* - 1o.
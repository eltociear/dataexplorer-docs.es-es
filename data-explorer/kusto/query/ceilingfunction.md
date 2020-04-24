---
title: ceiling() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe ceiling() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f2ecd043c43bb1af6530364d200d5dc9db640f95
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517262"
---
# <a name="ceiling"></a>ceiling()

Calcula el entero más pequeño mayor o igual que la expresión numérica especificada.

**Sintaxis**

`ceiling(`*X*`)`

**Argumentos**

* *x*: Un número real.

**Devuelve**

* El entero más pequeño mayor o igual que la expresión numérica especificada. 

**Ejemplos**

```kusto
print c1 = ceiling(-1.1), c2 = ceiling(0), c3 = ceiling(0.9)
```

|c1|c2|c3|
|---|---|---|
|-1|0|1|
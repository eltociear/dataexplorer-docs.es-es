---
title: isfinite() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isfinite() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5a17a39cce91fe039b2cf55cc5c98dba111cc334
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513607"
---
# <a name="isfinite"></a>isfinite()

Devuelve si la entrada es un valor finito (no es infinito ni NaN).

**Sintaxis**

`isfinite(`*X*`)`

**Argumentos**

* *x*: Un número real.

**Devuelve**

Un valor distinto de cero (true) si x es finito; y cero (falso) en caso contrario.

**Vea también**

* Para comprobar si value es null, consulte [isnull()](isnullfunction.md).
* Para comprobar si el valor es infinito, véase [isinf()](isinffunction.md).
* Para comprobar si el valor es NaN (Not-a-Number), consulte [isnan()](isnanfunction.md).

**Ejemplo**

```kusto
range x from -1 to 1 step 1
| extend y = 0.0
| extend div = 1.0*x/y
| extend isfinite=isfinite(div)
```

|x|y|div|isfinita|
|---|---|---|---|
|-1|0|-∞|0|
|0|0|NaN|0|
|1|0|∞|0|
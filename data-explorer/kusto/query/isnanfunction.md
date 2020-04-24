---
title: isnan() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isnan() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 123d9cd32d645bb1225983138973a17b6bb9ecf3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513573"
---
# <a name="isnan"></a>isnan()

Devuelve si la entrada es el valor Not-a-Number (NaN).  

**Sintaxis**

`isnan(`*X*`)`

**Argumentos**

* *x*: Un número real.

**Devuelve**

Un valor distinto de cero (true) si x es NaN; y cero (falso) en caso contrario.

**Vea también**

* Para comprobar si value es null, consulte [isnull()](isnullfunction.md).
* Para comprobar si el valor es finito, véase [isfinite()](isfinitefunction.md).
* Para comprobar si el valor es infinito, véase [isinf()](isinffunction.md).

**Ejemplo**

```kusto
range x from -1 to 1 step 1
| extend y = (-1*x) 
| extend div = 1.0*x/y
| extend isnan=isnan(div)
```

|x|y|div|isnan|
|---|---|---|---|
|-1|1|-1|0|
|0|0|NaN|1|
|1|-1|-1|0|
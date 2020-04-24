---
title: dayofyear() - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe dayofyear() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 41e7c5906da001e877dd9124124e126d729e886d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516225"
---
# <a name="dayofyear"></a>dayofyear()

Devuelve el número entero representa el número de día del año dado.

```kusto
dayofyear(datetime(2015-12-14))
```

**Sintaxis**

`dayofweek(`*a_date*`)`

**Argumentos**

* `a_date`: un valor `datetime`.

**Devuelve**

`day number`del año dado.
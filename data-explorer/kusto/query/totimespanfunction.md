---
title: totimespan() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe totimespan() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 504d4a74e8c1b58a8a97fd80d6c846fcf7e3f527
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505872"
---
# <a name="totimespan"></a>totimespan()

Convierte la entrada en escalar de intervalo de [tiempo.](./scalar-data-types/timespan.md)

```kusto
totimespan("0.00:01:00") == time(1min)
```

**Sintaxis**

`totimespan(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que se convertirá [en intervalo](./scalar-data-types/timespan.md)de tiempo . 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor de intervalo de [tiempo.](./scalar-data-types/timespan.md)
Si la conversión no se realiza correctamente, el resultado será null.
 
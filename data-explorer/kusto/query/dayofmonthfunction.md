---
title: dayofmonth() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe dayofmonth() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 791d85d4a8f89487e65ef68ecc605f907e63e1ea
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516344"
---
# <a name="dayofmonth"></a>dayofmonth()

Devuelve el número entero que representa el número de día del mes dado

```kusto
dayofmonth(datetime(2015-12-14)) == 14
```

**Sintaxis**

`dayofmonth(`*a_date*`)`

**Argumentos**

* `a_date`: un valor `datetime`.

**Devuelve**

`day number`del mes dado.
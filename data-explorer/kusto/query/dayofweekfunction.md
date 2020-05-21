---
title: DayOfWeek ()-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe DayOfWeek () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4e445a86b976f251de2beef4726c4840bcec8e44
ms.sourcegitcommit: ee90472a4f9d751d4049744d30e5082029c1b8fa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/21/2020
ms.locfileid: "83722038"
---
# <a name="dayofweek"></a>dayofweek()

Devuelve el número entero de días transcurridos desde el domingo anterior, como `timespan` .

```kusto
dayofweek(datetime(2015-12-14)) == 1d  // Monday
```

**Sintaxis**

`dayofweek(`*a_date*`)`

**Argumentos**

* `a_date`: un valor `datetime`.

**Devuelve**

El valor `timespan` desde medianoche al principio del domingo anterior, redondeado a un número entero de días.

**Ejemplos**

```kusto
dayofweek(datetime(1947-11-30 10:00:05))  // time(0.00:00:00), indicating Sunday
dayofweek(datetime(1970-05-11))           // time(1.00:00:00), indicating Monday
```

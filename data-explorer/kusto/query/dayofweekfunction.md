---
title: dayofweek() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe dayofweek() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 31d3f525653f6e0979229e4355cdec6cb76833f8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516310"
---
# <a name="dayofweek"></a>dayofweek()

Devuelve el número entero de días desde `timespan`el domingo anterior, como un archivo .

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
dayofweek(1947-11-29 10:00:05)  // time(6.00:00:00), indicating Saturday
dayofweek(1970-05-11)           // time(1.00:00:00), indicating Monday
```
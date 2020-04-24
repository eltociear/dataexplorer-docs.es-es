---
title: endofmonth() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe endofmonth() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ebab067c730e3cd61c84ae33eba4e49d7f0b0c0c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515868"
---
# <a name="endofmonth"></a>endofmonth()

Devuelve el final del mes que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

**Sintaxis**

`endofmonth(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de meses de desplazamiento a partir de la fecha de entrada (entero, predeterminado - 0).

**Devuelve**

Una fecha y hora que representa el final del mes para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project monthEnd = endofmonth(datetime(2017-01-01 10:10:17), offset) 
```

|monthEnd|
|---|
|2016-12-31 23:59:59.9999999|
|2017-01-31 23:59:59.9999999|
|2017-02-28 23:59:59.9999999|
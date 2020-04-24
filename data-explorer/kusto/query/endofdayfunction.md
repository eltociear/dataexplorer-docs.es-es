---
title: endofday() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe endofday() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a25f6b9c4ba660fbb8a50390d9d6920ff21caeb8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515885"
---
# <a name="endofday"></a>endofday()

Devuelve el final del día que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

**Sintaxis**

`endofday(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de días de desplazamiento desde la fecha de entrada (entero, predeterminado - 0).

**Devuelve**

Una fecha y hora que representa el final del día para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project dayEnd = endofday(datetime(2017-01-01 10:10:17), offset) 
```

|dayEnd|
|---|
|2016-12-31 23:59:59.9999999|
|2017-01-01 23:59:59.9999999|
|2017-01-02 23:59:59.9999999|
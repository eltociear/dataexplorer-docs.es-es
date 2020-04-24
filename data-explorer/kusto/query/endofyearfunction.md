---
title: endofyear() - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe endofyear() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d4a14d1cc42d5b9116e8a144e2b67fb74c1932ee
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515783"
---
# <a name="endofyear"></a>endofyear()

Devuelve el final del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

**Sintaxis**

`endofyear(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de años de desplazamiento a partir de la fecha de entrada (entero, predeterminado - 0).

**Devuelve**

Una fecha y hora que representa el final del año para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project yearEnd = endofyear(datetime(2017-01-01 10:10:17), offset) 
```

|yearEnd|
|---|
|2016-12-31 23:59:59.9999999|
|2017-12-31 23:59:59.9999999|
|2018-12-31 23:59:59.9999999|
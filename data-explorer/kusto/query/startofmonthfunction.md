---
title: startofmonth() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe startofmonth() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e220919f6b09dbcd2519c67f48148a6375395e7b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507249"
---
# <a name="startofmonth"></a>startofmonth()

Devuelve el inicio del mes que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

**Sintaxis**

`startofmonth(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de meses de desplazamiento a partir de la fecha de entrada (entero, predeterminado - 0).

**Devuelve**

Una fecha y hora que representa el inicio del mes para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project monthStart = startofmonth(datetime(2017-01-01 10:10:17), offset) 
```

|monthStart|
|---|
|2016-12-01 00:00:00.0000000|
|2017-01-01 00:00:00.0000000|
|2017-02-01 00:00:00.0000000|
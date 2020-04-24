---
title: startofweek() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe startofweek() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9705b586a0b8c5161b7d4c27735f644b6982c707
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507232"
---
# <a name="startofweek"></a>startofweek()

Devuelve el inicio de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

El inicio de la semana se considera un domingo.

**Sintaxis**

`startofweek(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de semanas de desplazamiento desde la fecha de entrada (entero, predeterminado - 0).

**Devuelve**

Una fecha y hora que representa el inicio de la semana para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project weekStart = startofweek(datetime(2017-01-01 10:10:17), offset) 
```

|weekStart|
|---|
|2016-12-25 00:00:00.0000000|
|2017-01-01 00:00:00.0000000|
|2017-01-08 00:00:00.0000000|
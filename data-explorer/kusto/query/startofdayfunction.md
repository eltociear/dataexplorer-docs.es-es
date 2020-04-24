---
title: startofday() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe startofday() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6517b50ca880085761212ae9037cee96d20a3269
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507283"
---
# <a name="startofday"></a>startofday()

Devuelve el inicio del día que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

**Sintaxis**

`startofday(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de días de desplazamiento desde la fecha de entrada (entero, predeterminado - 0). 

**Devuelve**

Una fecha y hora que representa el inicio del día para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project dayStart = startofday(datetime(2017-01-01 10:10:17), offset) 
```

|dayStart|
|---|
|2016-12-31 00:00:00.0000000|
|2017-01-01 00:00:00.0000000|
|2017-01-02 00:00:00.0000000|
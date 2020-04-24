---
title: startofyear() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe startofyear() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f91c749cc3833954d902eb4ebd7e230e32e3a991
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507215"
---
# <a name="startofyear"></a>startofyear()

Devuelve el inicio del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

**Sintaxis**

`startofyear(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de años de desplazamiento a partir de la fecha de entrada (entero, predeterminado - 0). 

**Devuelve**

Una fecha y hora que representa el inicio del año para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project yearStart = startofyear(datetime(2017-01-01 10:10:17), offset) 
```

|yearStart|
|---|
|2016-01-01 00:00:00.0000000|
|2017-01-01 00:00:00.0000000|
|2018-01-01 00:00:00.0000000|
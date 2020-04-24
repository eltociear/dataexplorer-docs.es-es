---
title: monthofyear() - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe monthofyear() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6ada34d3e6f6c905a1acfb550b02af3dab550fb8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512349"
---
# <a name="monthofyear"></a>monthofyear()

Devuelve el número entero representa el número de mes del año dado.

Otro alias: getmonth()

```kusto
monthofyear(datetime("2015-12-14"))
```

**Sintaxis**

`monthofyear(`*a_date*`)`

**Argumentos**

* `a_date`: un valor `datetime`.

**Devuelve**

`month number`del año dado.
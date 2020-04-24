---
title: hourofday() - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe hourofday() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 08fdb45af5eec7f71d491725ea58a7d72c06371b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514083"
---
# <a name="hourofday"></a>hourofday()

Devuelve el número entero que representa el número de hora de la fecha dada

```kusto
hourofday(datetime(2015-12-14 18:54)) == 18
```

**Sintaxis**

`hourofday(`*a_date*`)`

**Argumentos**

* `a_date`: un valor `datetime`.

**Devuelve**

`hour number`del día (0-23).
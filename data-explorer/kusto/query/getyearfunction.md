---
title: getyear() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe getyear() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a0e9d4c3e8c793f7775154261febc11e58082132
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514338"
---
# <a name="getyear"></a>getyear()

Devuelve la parte `datetime` del año del argumento.

**Ejemplo**

```kusto
T
| extend year = getyear(datetime(2015-10-12))
// year == 2015
```
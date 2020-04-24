---
title: isnotnull() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isnotnull() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 8be57f2f7484081ac316a8af6fea252a60f895a4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513454"
---
# <a name="isnotnull"></a>isnotnull()

Devuelve `true` si el argumento no es null.

**Sintaxis**

`isnotnull(`[*valor*]`)`

`notnull(`[*valor*] `)` - alias para`isnotnull`

**Ejemplo**

```kusto
T | where isnotnull(PossiblyNull) | count
```

Tenga en cuenta que hay otras formas de conseguir este efecto:

```kusto
T | summarize count(PossiblyNull)
```
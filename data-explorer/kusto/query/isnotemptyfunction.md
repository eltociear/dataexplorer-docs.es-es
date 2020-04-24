---
title: isnotempty() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isnotempty() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 14111be0fc0247dd151ef7454121e6b90a32ff0d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513539"
---
# <a name="isnotempty"></a>isnotempty()

Devuelve `true` si el argumento no es una cadena vacía ni es un valor null.

```kusto
isnotempty("") == false
```

**Sintaxis**

`isnotempty(`[*valor*]`)`

`notempty(`[*valor*] `)` -- alias de`isnotempty`
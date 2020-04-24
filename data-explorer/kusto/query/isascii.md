---
title: isascii() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isascii() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: daba0f4015a4847155309964f8ac0909ff4bc9d0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513726"
---
# <a name="isascii"></a>isascii()

Devuelve `true` si el argumento es una cadena ascii válida.
    
```kusto
isascii("some string") == true
```

**Sintaxis**

`isascii(`[*valor*]`)`

**Devuelve**

Indica si el argumento es una cadena ascii válida.

**Ejemplo**

```kusto
T
| where isascii(fieldName)
| count
```
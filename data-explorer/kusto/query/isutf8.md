---
title: isutf8() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isutf8() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 619fb90b72fed8ec0e10fe05ddc3c6df6ff1386e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513403"
---
# <a name="isutf8"></a>isutf8()

Devuelve `true` si el argumento es una cadena utf8 válida.
    
```kusto
isutf8("some string") == true
```

**Sintaxis**

`isutf8(`[*valor*]`)`

**Devuelve**

Indica si el argumento es una cadena utf8 válida.

**Ejemplo**

```kusto
T
| where isutf8(fieldName)
| count
```
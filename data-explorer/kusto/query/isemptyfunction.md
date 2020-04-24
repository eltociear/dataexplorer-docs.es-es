---
title: isempty() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isempty() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2cb0e53aa16257398c20661c31494ca9dda17c1e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513624"
---
# <a name="isempty"></a>isempty()

Devuelve `true` si el argumento es una cadena vacía o es null.
    
```kusto
isempty("") == true
```

**Sintaxis**

`isempty(`[*valor*]`)`

**Devuelve**

Indica si el argumento es una cadena vacía o isnull.

|x|isempty(x)
|---|---
| "" | true
|"x" | False
|parsejson("")|true
|parsejson("[]")|False
|parsejson("{}")|False

**Ejemplo**

```kusto
T
| where isempty(fieldName)
| count
```
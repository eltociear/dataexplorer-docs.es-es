---
title: not() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe not() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 539e409a9e922afc390b097c863146b7fc30d7b3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512043"
---
# <a name="not"></a>not()

Invierte el valor `bool` de su argumento.

```kusto
not(false) == true
```

**Sintaxis**

`not(`*Expr*`)`

**Argumentos**

* *expr*: `bool` Una expresión que se va a invertir.

**Devuelve**

Devuelve el valor lógico `bool` invertido de su argumento.
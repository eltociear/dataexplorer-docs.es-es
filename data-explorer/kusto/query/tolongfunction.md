---
title: tolong() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe tolong() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bd354a39c048631ab98390c74cb7cd78ee5376b2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506093"
---
# <a name="tolong"></a>tolong()

Convierte la entrada en una representación de número larga (firmada de 64 bits).

```kusto
tolong("123") == 123
```

**Sintaxis**

`tolong(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que se convertirá en long. 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número largo.
Si la conversión no `null`se realiza correctamente, el resultado será .
 
*Nota*: Prefiere usar [long()](./scalar-data-types/long.md) cuando sea posible.
---
title: tobool() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe tobool() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3e1867c99ae241b3f7e09ab8ee873d5ae5d374e0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506365"
---
# <a name="tobool"></a>tobool()

Convierte la entrada en representación booleana (8 bits firmada).

```kusto
tobool("true") == true
tobool("false") == false
tobool(1) == true
tobool(123) == true
```

**Sintaxis**

`tobool(`*Expr*`)`
`)` Expr (alias)*Expr* `toboolean(`

**Argumentos**

* *Expr*: Expresión que se convertirá en booleana. 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un booleano.
Si la conversión no `null`se realiza correctamente, el resultado será .
 
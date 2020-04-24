---
title: todecimal() - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe todecimal() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2d5ac70dfe71f80c3963292516e1b8516a297875
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506314"
---
# <a name="todecimal"></a>todecimal()

Convierte la entrada en representación de números decimales.

```kusto
todecimal("123.45678") == decimal(123.45678)
```

**Sintaxis**

`todecimal(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que se convertirá a decimal. 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número decimal.
Si la conversión no `null`se realiza correctamente, el resultado será .
 
*Nota*: Prefiere usar [real()](./scalar-data-types/real.md) cuando sea posible.
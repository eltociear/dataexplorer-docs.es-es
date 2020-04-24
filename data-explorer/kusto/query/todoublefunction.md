---
title: todouble()/toreal() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe todouble()/toreal() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: eb9c976f1646f71fcf8b345899037461f58f4ef0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506331"
---
# <a name="todoubletoreal"></a>todouble()/toreal()

Convierte la entrada en un `real`valor de tipo . (`todouble()` `toreal()` y son sinónimos.)

```kusto
toreal("123.4") == 123.4
```

**Sintaxis**

`toreal(`*Expr*`)`
`todouble(`*Expr*`)`

**Argumentos**

* *Expr*: una expresión cuyo valor se `real`convertirá en un valor de tipo .

**Devuelve**

Si la conversión se realiza correctamente, el resultado es un valor de tipo `real`.
Si la conversión no se `real(null)`realiza correctamente, el resultado es el valor .

*Nota*: Prefiere usar [double() o real()](./scalar-data-types/real.md) cuando sea posible.
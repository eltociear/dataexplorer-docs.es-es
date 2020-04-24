---
title: toint() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe toint() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 570e13dc816c8a7e6d5baa488912fd8def5d2883
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506110"
---
# <a name="toint"></a>toint()

Convierte la entrada en una representación numérica de enteros (con signo de 32 bits).

```kusto
toint("123") == 123
```

**Sintaxis**

`toint(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que se convertirá en entero. 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número entero.
Si la conversión no `null`se realiza correctamente, el resultado será .
 
*Nota*: Prefiere usar [int()](./scalar-data-types/int.md) cuando sea posible.
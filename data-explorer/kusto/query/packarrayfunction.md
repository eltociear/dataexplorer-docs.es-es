---
title: pack_array() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe pack_array() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: ad13efd6b0743a3c092b2859ea032c3731ffdf1b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511873"
---
# <a name="pack_array"></a>pack_array()

Empaqueta todos los valores de entrada en una matriz dinámica.

**Sintaxis**

`pack_array(`*Expr1*`[`` *Expr2*]`, )'

**Argumentos**

* *Expr1... N*: Expresiones de entrada que se empaquetarán en una matriz dinámica.

**Devuelve**

Matriz dinámica que incluye los valores de Expr1, Expr2, ... , ExprN.

**Ejemplo**

```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project pack_array(x,y,z)
```

|Column1|
|---|
|[1,2,4]|
|[2,4,8]|
|[3,6,12]|

```kusto
range x from 1 to 3 step 1
| extend y = tostring(x * 2)
| extend z = (x * 2) * 1s
| project pack_array(x,y,z)
```

|Column1|
|---|
|[1,"2","00:00:02"]|
|[2,"4","00:00:04"]|
|[3,"6","00:00:06"]|
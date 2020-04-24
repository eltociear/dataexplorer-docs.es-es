---
title: todatetime() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe todatetime() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: abe3852195b1a79ab5c86176698099ed6e7ff7af
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506348"
---
# <a name="todatetime"></a>todatetime()

Convierte la entrada en escalar [datetime.](./scalar-data-types/datetime.md)

```kusto
todatetime("2015-12-24") == datetime(2015-12-24)
```

**Sintaxis**

`todatetime(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que se convertirá a [datetime](./scalar-data-types/datetime.md). 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor [datetime.](./scalar-data-types/datetime.md)
Si la conversión no se realiza correctamente, el resultado será null.
 
*Nota*: Prefiere usar [datetime()](./scalar-data-types/datetime.md) cuando sea posible.
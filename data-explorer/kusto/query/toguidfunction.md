---
title: toguid() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe toguid() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: fb1df5fe91b75e5d3b7d1a9f40b8b3079cac9944
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506144"
---
# <a name="toguid"></a>toguid()

Convierte la [`guid`](./scalar-data-types/guid.md) entrada en representación.

```kusto
toguid("70fc66f7-8279-44fc-9092-d364d70fce44") == guid("70fc66f7-8279-44fc-9092-d364d70fce44")
```

**Sintaxis**

`toguid(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que [`guid`](./scalar-data-types/guid.md) se convertirá en escalar. 

**Devuelve**

Si la conversión se [`guid`](./scalar-data-types/guid.md) realiza correctamente, el resultado será un escalar.
Si la conversión no `null`se realiza correctamente, el resultado será .

*Nota*: Prefiere usar [guid()](./scalar-data-types/guid.md) cuando sea posible.
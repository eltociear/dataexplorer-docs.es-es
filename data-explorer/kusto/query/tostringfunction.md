---
title: tostring() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe tostring() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 51aaf90b60653a648457dc00200168aec7fbefd9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505889"
---
# <a name="tostring"></a>tostring()

Convierte la entrada en una representación de cadena.

```kusto
tostring(123) == "123"
```

**Sintaxis**

`tostring(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión que se convertirá en cadena. 

**Devuelve**

Si el valor *Expr* es no nulo, el resultado será una representación de cadena de *Expr*.
Si el valor *de Expr* es null, el resultado será una cadena vacía.
 
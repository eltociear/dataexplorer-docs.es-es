---
title: 'ToString (): Explorador de datos de Azure'
description: En este artículo se describe ToString () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 634f54533e83575139d8399124cc068af56d8574
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741668"
---
# <a name="tostring"></a>tostring()

Convierte la entrada en una representación de cadena.

```kusto
tostring(123) == "123"
```

**Sintaxis**

`tostring(`*`Expr`*`)`

**Argumentos**

* *`Expr`*: Expresión que se convertirá en una cadena. 

**Devuelve**

Si el *`Expr`* valor no es null, el resultado será una representación de cadena de *`Expr`*.
Si el *`Expr`* valor es null, el resultado será una cadena vacía.
 
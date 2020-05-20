---
title: 'Toint ((): Azure Explorador de datos'
description: En este artículo se describe Toint (() en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7f0ede908be2689165f641038b2b6f699c0eb543
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550612"
---
# <a name="toint"></a>toint()

Convierte la entrada en la representación de número entero (con signo 32).

```kusto
toint("123") == 123s
```

**Sintaxis**

`toint(`*Argumento*`)`

**Argumentos**

* *Expr*: expresión que se convertirá en un entero. 

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un entero.
Si la conversión no se realiza correctamente, el resultado será `null` .
 
*Nota*: prefiere usar [int ()](./scalar-data-types/int.md) siempre que sea posible.
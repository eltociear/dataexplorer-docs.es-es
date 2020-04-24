---
title: tohex() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe tohex() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 402a0923d4fe760e97fa6098ad955b6c24a5d5ee
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506127"
---
# <a name="tohex"></a>tohex()

Convierte la entrada en una cadena hexadecimal.

```kusto
tohex(256) == '100'
tohex(-256) == 'ffffffffffffff00' // 64-bit 2's complement of -256
tohex(toint(-256), 8) == 'ffffff00' // 32-bit 2's complement of -256
tohex(256, 8) == '00000100'
tohex(256, 2) == '100' // Exceeds min length of 2, so min length is ignored.
```

**Sintaxis**

`tohex(`*Expr*`, [`` *MinLength*]`, )'

**Argumentos**

* *Expr*: valor int o long que se convertirá en una cadena hexadecimal.  No se admiten otros tipos.
* *MinLength*: valor numérico que representa el número de caracteres iniciales que se incluirán en la salida.  Se admiten valores entre 1 y 16, los valores mayores que 16 se truncarán a 16.  Si la cadena es más larga que minLength sin caracteres iniciales, minLength se omite eficazmente.  Los números negativos solo se pueden representar como mínimo por su tamaño de datos subyacente, por lo que para un int (32 bits) la minLength será como mínimo 8, durante un largo (64 bits) será como mínimo 16.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor de cadena.
Si la conversión no se realiza correctamente, el resultado será null.
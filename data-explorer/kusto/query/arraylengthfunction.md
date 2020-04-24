---
title: array_length() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_length() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0bb1daeba6d24f8bd7326fcd0b8c17f06003e30b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518605"
---
# <a name="array_length"></a>array_length()

Calcula el número de elementos de una matriz dinámica.

**Sintaxis**

`array_length(`*Matriz*`)`

**Argumentos**

* *matriz*: `dynamic` Un valor.

**Devuelve**

El número de elementos de *array* o `null` si *array* no es una matriz.

**Ejemplos**

```kusto
print array_length(parse_json('[1, 2, 3, "four"]')) == 4

print array_length(parse_json('[8]')) == 1

print array_length(parse_json('[{}]')) == 1

print array_length(parse_json('[]')) == 0

print array_length(parse_json('{}')) == null

print array_length(parse_json('21')) == null
```
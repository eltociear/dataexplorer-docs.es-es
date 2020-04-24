---
title: 'operador principal: Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador principal en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bb1d24dd0cd1dfeecb1925e474eb77e8cb86121f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505923"
---
# <a name="top-operator"></a>Operador top

Devuelve los primeros *N* registros ordenados por las columnas especificadas.

```kusto
T | top 5 by Name desc nulls last
```

**Sintaxis**

*T* `| top` *NumberOfRows* `by` `asc` | `desc``nulls first` | `nulls last` *Expresión* [ ] [ ]

**Argumentos**

* *NumberOfRows*: el número de filas de *T* que se van a devolver. Puede especificar cualquier expresión numérica.
* *Expresión*: Una expresión escalar por la que ordenar. El tipo de los valores tiene que ser numérico, fecha, hora o cadena.
* `asc` o `desc` (valor predeterminado) puede aparecer para controlar si la selección se hace realmente desde la parte "superior" o desde la parte "inferior" del intervalo.
* `nulls first`(el valor `asc` predeterminado `nulls last` para order) `desc` o (el valor predeterminado para order) puede parecer controlar si los valores nulos estarán al principio o al final del intervalo.


**Sugerencias**

* `top 5 by name`es equivalente a `sort by name | take 5` la expresión tanto desde perspectivas semánticas como de rendimiento..
* Utilice el operador [anidado superior](topnestedoperator.md) para producir resultados superiores jerárquicos (anidados).
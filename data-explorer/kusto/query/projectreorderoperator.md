---
title: 'operador de reorden de proyectos: Explorador de azure Data Explorer ( Project Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de reorden de proyecto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bf56a69891f83aaabc12dbbcd8f758ecb963b493
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510853"
---
# <a name="project-reorder-operator"></a>Operador project-reorder

Reordena las columnas en la salida del resultado.

```kusto
T | project-reorder Col2, Col1, Col* asc
```

**Sintaxis**

*T* `| project-reorder` *ColumnNameOrPattern* [`asc`|`desc`] [`,` ...]

**Argumentos**

* *T*: La tabla de entrada.
* *ColumnNameOrPattern:* El nombre del patrón de caracteres comodín de columna o columna agregado a la salida.
* Para patrones de caracteres comodín: especificar `asc` u `desc` ordenar columnas utilizando sus nombres en orden ascendente o descendente. Si `asc` `desc` se especifica o no, el orden viene determinado por las columnas coincidentes tal como aparecen en la tabla de origen.

**Devuelve**

Tabla que contiene columnas en el orden especificado por los argumentos del operador. `project-reorder`no cambia el nombre ni quita columnas de la tabla, por lo tanto, todas las columnas que existían en la tabla de origen aparecen en la tabla de resultados.

**Notas**

- En la coincidencia ambigua *ColumnNameOrPattern,* la columna aparece en la primera posición que coincide con el patrón.
- Especificar columnas para `project-reorder` el es opcional. Las columnas que no se especifican aparecen explícitamente como las últimas columnas de la tabla de salida.

* Se [`project-away`](projectawayoperator.md) utiliza para quitar columnas.
* Se [`project-rename`](projectrenameoperator.md) utiliza para cambiar el nombre de las columnas.


**Ejemplos**

Reordene una tabla con tres columnas (a, b, c) para que la segunda columna (b) aparezca primero.

```kusto
print a='a', b='b', c='c'
|  project-reorder b
```

|b|a|c|
|---|---|---|
|b|a|c|

Reordene las columnas de una `a` tabla para que las columnas que comiencen con aparezcan antes que otras columnas.

```kusto
print b = 'b', a2='a2', a3='a3', a1='a1'
|  project-reorder a* asc
```

|a1|a2|a3|b|
|---|---|---|---|
|a1|a2|a3|b|
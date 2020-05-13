---
title: proyecto-reordenar operador-Azure Explorador de datos
description: En este artículo se describe el operador de reordenación del proyecto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 74acab0dc4f0fbdaf7c77e609db3e41f875f2cad
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373151"
---
# <a name="project-reorder-operator"></a>Operador project-reorder

Reordena las columnas en la salida de resultados.

```kusto
T | project-reorder Col2, Col1, Col* asc
```

**Sintaxis**

*T* `| project-reorder` *ColumnNameOrPattern* [ `asc` | `desc` ] [ `,` ...]

**Argumentos**

* *T*: la tabla de entrada.
* *ColumnNameOrPattern:* Nombre del patrón de caracteres comodín de columna o columna agregado a la salida.
* Para los patrones de caracteres comodín: especificar `asc` o `desc` ordenar columnas usando sus nombres en orden ascendente o descendente. Si `asc` `desc` no se especifica o, el orden viene determinado por las columnas coincidentes tal y como aparecen en la tabla de origen.

**Devuelve**

Tabla que contiene las columnas en el orden especificado por los argumentos del operador. `project-reorder`no cambia el nombre ni quita columnas de la tabla, por lo que todas las columnas que existían en la tabla de origen aparecen en la tabla de resultados.

**Notas**

- En la coincidencia *ColumnNameOrPattern* ambigua, la columna aparece en la primera posición que coincide con el patrón.
- Especificar columnas para `project-reorder` es opcional. Las columnas que no se especifican aparecen explícitamente como las últimas columnas de la tabla de salida.

* Use [`project-away`](projectawayoperator.md) para quitar columnas.
* Use [`project-rename`](projectrenameoperator.md) para cambiar el nombre de las columnas.


**Ejemplos**

Reordene una tabla con tres columnas (a, b, c), por lo que la segunda columna (b) aparecerá en primer lugar.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print a='a', b='b', c='c'
|  project-reorder b
```

|b|a|c|
|---|---|---|
|b|a|c|

Reordene las columnas de una tabla para que las columnas que comienzan por `a` aparezcan antes que otras columnas.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print b = 'b', a2='a2', a3='a3', a1='a1'
|  project-reorder a* asc
```

|a1|R2|a3|b|
|---|---|---|---|
|a1|R2|a3|b|

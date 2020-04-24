---
title: 'operador de alejación del proyecto: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe el operador de alejación del proyecto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 38ec57e9659458ef34117e4a380c756310db218b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510955"
---
# <a name="project-away-operator"></a>Operador project-away

Seleccione qué columnas de la entrada excluir de la salida

```kusto
T | project-away price, quantity, zz*
```

El orden de las columnas en el resultado viene determinado por su orden original en la tabla. Solo se quitan las columnas que se especificaron como argumentos. Las otras columnas se incluyen en el resultado.  (Consulte también `project`).

**Sintaxis**

*T* `| project-away` *ColumnNameOrPattern* [`,` ...]

**Argumentos**

* *T*: La tabla de entrada
* *ColumnNameOrPattern:* El nombre de la columna o columna comodín-patrón que se va a quitar de la salida.

**Devuelve**

Una tabla con columnas que no se denominaron como argumentos. Contiene el mismo número de filas que la tabla de entrada.

**Sugerencias**

* Utilícelo [`project-rename`](projectrenameoperator.md) si su intención es cambiar el nombre de las columnas.
* Utilícelo [`project-reorder`](projectreorderoperator.md) si su intención es reordenar las columnas.

* Puede `project-away` las columnas que están presentes en la tabla original o que se calcularon como parte de la consulta.


**Ejemplos**

La tabla de entrada `T` tiene tres columnas de tipo `long`: `A`, `B` y `C`.

```kusto
datatable(A:long, B:long, C:long) [1, 2, 3]
| project-away C    // Removes column C from the output
```

|Un|B|
|---|---|
|1|2|

Eliminación de columnas que comienzan por 'a'.

```kusto
print  a2='a2', b = 'b', a3='a3', a1='a1'
|  project-away a* 
```

|b|
|---|
|b|


---
title: 'operador de proyección: Azure Explorador de datos'
description: En este artículo se describe el operador de ubicación de proyecto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 444710775af405cc63193e0205e573b2ea77de3a
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373183"
---
# <a name="project-away-operator"></a>Operador project-away

Seleccionar las columnas de la entrada que se deben excluir de la salida

```kusto
T | project-away price, quantity, zz*
```

El orden de las columnas en el resultado viene determinado por su orden original en la tabla. Solo se quitan las columnas especificadas como argumentos. Las demás columnas se incluyen en el resultado.  (Consulte también `project`).

**Sintaxis**

*T* `| project-away` *ColumnNameOrPattern* [ `,` ...]

**Argumentos**

* *T*: la tabla de entrada
* *ColumnNameOrPattern:* Nombre del patrón de caracteres comodín de columna o columna que se va a quitar de la salida.

**Devuelve**

Una tabla con columnas que no se denominaban argumentos. Contiene el mismo número de filas que la tabla de entrada.

**Sugerencias**

* Use [`project-rename`](projectrenameoperator.md) si su intención es cambiar el nombre de las columnas.
* Use [`project-reorder`](projectreorderoperator.md) si su intención es reordenar las columnas.

* Puede `project-away` usar cualquier columna que esté presente en la tabla original o que se hayan calculado como parte de la consulta.


**Ejemplos**

La tabla de entrada `T` tiene tres columnas de tipo `long`: `A`, `B` y `C`.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(A:long, B:long, C:long) [1, 2, 3]
| project-away C    // Removes column C from the output
```

|Un|B|
|---|---|
|1|2|

Quitando columnas que comienzan por ' a '.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print  a2='a2', b = 'b', a3='a3', a1='a1'
|  project-away a* 
```

|b|
|---|
|b|


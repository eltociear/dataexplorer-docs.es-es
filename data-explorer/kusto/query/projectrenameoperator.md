---
title: Project-Rename Operator-Azure Explorador de datos
description: En este artículo se describe el operador de cambio de nombre de proyecto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 68581cfe4b3828823ced7d4704eb08df5d2aefa7
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373169"
---
# <a name="project-rename-operator"></a>Operador project-rename

Cambia el nombre de las columnas en la salida de resultados.

```kusto
T | project-rename new_column_name = column_name
```

**Sintaxis**

*T* `| project-rename` *NewColumnName*  =  *ExistingColumnName* [ `,` ...]

**Argumentos**

* *T*: la tabla de entrada.
* *NewColumnName:* El nuevo nombre de una columna. 
* *ExistingColumnName:* Nombre existente de una columna. 

**Devuelve**

Tabla que tiene las columnas en el mismo orden que en una tabla existente, con las columnas cuyo nombre se ha cambiado.


**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print a='a', b='b', c='c'
|  project-rename new_b=b, new_a=a
```

|new_a|new_b|c|
|---|---|---|
|a|b|c|

---
title: 'operador de cambio de nombre de proyecto: Explorador de azure Data Explorer ( Project Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de cambio de nombre de proyecto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1f7fbf2efbfd3ed1dfac9129ec21f5a13c51481c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510870"
---
# <a name="project-rename-operator"></a>Operador project-rename

Cambia el nombre de las columnas en la salida del resultado.

```kusto
T | project-rename new_column_name = column_name
```

**Sintaxis**

*T* `| project-rename` *NewColumnName* = *ExistingColumnName* [`,` ...]

**Argumentos**

* *T*: La tabla de entrada.
* *NewColumnName:* El nuevo nombre de una columna. 
* *ExistingColumnName:* El nombre existente de una columna. 

**Devuelve**

Una tabla que tiene las columnas en el mismo orden que en una tabla existente, con columnas renombradas.


**Ejemplos**

```kusto
print a='a', b='b', c='c'
|  project-rename new_b=b, new_a=a
```

|new_a|new_b|c|
|---|---|---|
|a|b|c|
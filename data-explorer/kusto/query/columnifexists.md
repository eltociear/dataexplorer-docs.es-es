---
title: column_ifexists() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe column_ifexists() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c4f7d4a2114aa2b9b3bb8ae3e951306a3ffb725e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517177"
---
# <a name="column_ifexists"></a>column_ifexists()

Toma un nombre de columna como una cadena y un valor predeterminado. Devuelve una referencia a la columna si existe, de lo contrario - devuelve el valor predeterminado.

**Sintaxis**

`column_ifexists(`*columnName*`, `*defaultValue*)

**Argumentos**

* *columnName*: El nombre de la columna
* *defaultValue*: el valor que se usará si la columna no existe en el contexto en el que se usó la función.
                  Este valor puede ser cualquier expresión escalar (por ejemplo, una referencia a otra columna).

**Devuelve**

Si *columnName* existe, la columna a la que hace referencia. De lo contrario , *defaultValue*.

**Ejemplos**

```kusto
.create function with (docstring = "Wraps a table query that allows querying the table even if columnName doesn't exist ", folder="My Functions")
ColumnOrDefault(tableName:string, columnName:string)
{
    // There's no column "Capital" in "StormEvents", therefore, the State column will be used instead
    table(tableName) | project column_ifexists(columnName, State)
}


ColumnOrDefault("StormEvents", "Captial");
```
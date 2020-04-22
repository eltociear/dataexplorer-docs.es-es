---
title: operador datatable - Explorador de datos de Azure ( Data Explorer) Microsoft Docs
description: En este artículo se describe el operador datatable en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 182f8030e3263ee5bf6bee4ca7444b0d5596e7d6
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765383"
---
# <a name="datatable-operator"></a>Operador datatable

Devuelve una tabla cuyo esquema y valores se definen en la propia consulta.

Tenga en cuenta que este operador no tiene una entrada de canalización.

**Sintaxis**

`datatable``(` *ColumnName* `:` *ColumnType* [`,` ...] `)` *ScalarValue* `,` *ScalarValue* ScalarValue [ ScalarValue ...] `[``]`

**Argumentos**

::: zone pivot="azuredataexplorer"

* *ColumnName*, *ColumnType*: Definen el esquema de la tabla. La sintaxis utilizada es precisamente la misma que la sintaxis utilizada al definir una tabla (consulte [tabla .create](../management/create-table-command.md)).
* *ScalarValue*: Un valor escalar constante para insertar en la tabla. El número de valores debe ser un múltiplo entero de las columnas de la tabla y el valor *n*'th debe tener un tipo que corresponda a la columna *n* % *NumColumns*.

::: zone-end

::: zone pivot="azuremonitor"

* *ColumnName*, *ColumnType*: Definen el esquema de la tabla.
* *ScalarValue*: Un valor escalar constante para insertar en la tabla. El número de valores debe ser un múltiplo entero de las columnas de la tabla y el valor *n*'th debe tener un tipo que corresponda a la columna *n* % *NumColumns*.

::: zone-end

**Devuelve**

Este operador devuelve una tabla de datos del esquema y los datos especificados.

**Ejemplo**

```kusto
datatable (Date:datetime, Event:string)
    [datetime(1910-06-11), "Born",
     datetime(1930-01-01), "Enters Ecole Navale",
     datetime(1953-01-01), "Published first book",
     datetime(1997-06-25), "Died"]
| where strlen(Event) > 4
```

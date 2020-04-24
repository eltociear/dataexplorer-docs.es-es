---
title: Tablas- Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las tablas del Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: fd47a8f59c716148dfc5f89ac1d4c7aca45a8e9c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509289"
---
# <a name="tables"></a>Tablas

Las tablas son entidades con nombre que contienen datos. Una tabla tiene un conjunto ordenado de [columnas](./columns.md)y cero o más filas de datos, cada fila contiene un valor de datos para cada una de las columnas de la tabla. El orden de las filas de la tabla es desconocido y en general no afecta a las consultas, excepto para algunos operadores tabulares (como el [operador superior)](../topoperator.md)que están inherentemente indeterminados.

Las tablas ocupan el mismo espacio de nombres que las [funciones almacenadas,](./stored-functions.md)por lo que si una función almacenada y una tabla tienen el mismo nombre, se elegirá la función almacenada.

**Notas**  

* Los nombres de tabla distinguen mayúsculas de minúsculas.
* Los nombres de tabla siguen las reglas para los nombres de [entidad.](./entity-names.md)
* El límite máximo de tablas por base de datos es de 10.000.


Los detalles sobre cómo crear y administrar tablas se pueden encontrar en la administración de [tablas](../../management/tables.md)

## <a name="table-references"></a>Referencias de tabla

La forma más sencilla de hacer referencia a una tabla es utilizando su nombre. Esto se puede hacer para todas las tablas que se encuentran en la base de datos en contexto. Así, por ejemplo, la siguiente consulta cuenta los `StormEvents` registros de la tabla de la base de datos actual:

```kusto
StormEvents
| count
```

Tenga en cuenta que una forma equivalente de escribir la consulta anterior es escapando al nombre de la tabla:

```kusto
["StormEvents"]
| count
```

También se puede hacer referencia a las tablas observando explícitamente la base de datos (o la base de datos y el clúster) en las que se encuentran, lo que permite crear consultas que combinan datos de varias bases de datos y clústeres. Por ejemplo, la siguiente consulta funcionará con cualquier base de datos en contexto, siempre y cuando el autor de la llamada tenga acceso a la base de datos de destino:

```kusto
cluster("https://help.kusto.windows.net").database("Samples").StormEvents
| count
```

También es posible hacer referencia a una tabla utilizando la [función especial table(),](../tablefunction.md)siempre que el argumento de esa función se evalúe como una constante. Por ejemplo:

```kusto
let counter=(TableName:string) { table(TableName) | count };
counter("StormEvents")
```
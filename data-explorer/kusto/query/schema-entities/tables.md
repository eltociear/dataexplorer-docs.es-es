---
title: 'Tablas: Azure Explorador de datos'
description: En este artículo se describen las tablas de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: f51e05abac44b85ab328e7df5645eeab51d2a274
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550629"
---
# <a name="tables"></a>Tablas

Las tablas son entidades con nombre que contienen datos. Una tabla tiene un conjunto ordenado de [columnas](./columns.md)y cero o más filas de datos. Cada fila contiene un valor de datos para cada una de las columnas de la tabla. Se desconoce el orden de las filas de la tabla y no afecta a las consultas generales, excepto en el caso de algunos operadores tabulares (como el [operador Top](../topoperator.md)) que no se determinan de forma inherente.

Las tablas ocupan el mismo espacio de nombres que [las funciones almacenadas](./stored-functions.md).
Si una función almacenada y una tabla tienen el mismo nombre, se elegirá la función almacenada.

**Notas**  

* Los nombres de tabla distinguen mayúsculas de minúsculas.
* Los nombres de tabla siguen las reglas para [los nombres de entidad](./entity-names.md).
* El límite máximo de tablas por base de datos es 10.000.


Puede encontrar información sobre cómo crear y administrar tablas en administración de [tablas](../../management/tables.md) .

## <a name="table-references"></a>Referencias de tabla

La manera más sencilla de hacer referencia a una tabla es mediante su nombre. Esta referencia se puede hacer para todas las tablas que se encuentran en la base de datos en contexto. Por ejemplo, la consulta siguiente cuenta los registros de la tabla de la base de datos actual `StormEvents` :

```kusto
StormEvents
| count
```

Una manera equivalente de escribir la consulta anterior es escapar del nombre de la tabla:

```kusto
["StormEvents"]
| count
```

También se puede hacer referencia a las tablas indicando explícitamente la base de datos (o la base de datos y el clúster) en que se encuentran. Después, puede crear consultas que combinen datos de varias bases de datos y clústeres. Por ejemplo, la siguiente consulta funcionará con cualquier base de datos en contexto, siempre que el autor de la llamada tenga acceso a la base de datos de destino:

```kusto
cluster("https://help.kusto.windows.net").database("Samples").StormEvents
| count
```

También es posible hacer referencia a una tabla mediante la [función especial Table ()](../tablefunction.md), siempre que el argumento de esa función se evalúe como una constante. Por ejemplo:

```kusto
let counter=(TableName:string) { table(TableName) | count };
counter("StormEvents")
```
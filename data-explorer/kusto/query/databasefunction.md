---
title: 'Database () (función de ámbito): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la base de datos () (función de ámbito) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 03753cf7dcabb7637335d3dc71f0b9bca773e710
ms.sourcegitcommit: fbe298e88542c0dcea0f491bb53ac427f850f729
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/24/2020
ms.locfileid: "82030453"
---
# <a name="database-scope-function"></a>Database () (función SCOPE)

::: zone pivot="azuredataexplorer"

Cambia la referencia de la consulta a una base de datos concreta dentro del ámbito del clúster. 

```kusto
database('Sample').StormEvents
cluster('help').database('Sample').StormEvents
```

**Sintaxis**

`database(`*stringConstant*`)`

**Argumentos**

* *stringConstant*: nombre de la base de datos a la que se hace referencia. La base `DatabaseName` de datos identificada `PrettyName`puede ser o. El argumento debe ser _constante_ antes de la ejecución de la consulta, es decir, no puede proviene de la evaluación de la subconsulta.

**Notas**

* Para tener acceso a un clúster remoto y a una base de datos remota, consulte función de ámbito [cluster ()](clusterfunction.md) .
* Más información sobre las consultas entre clústeres y entre bases de datos disponibles [aquí](cross-cluster-or-database-queries.md)

## <a name="examples"></a>Ejemplos

### <a name="use-database-to-access-table-of-other-database"></a>Use la base de datos () para tener acceso a la tabla de otras bases de datos. 

```kusto
database('Samples').StormEvents | count
```

|Count|
|---|
|59066|

### <a name="use-database-inside-let-statements"></a>Usar la base de datos () dentro de las instrucciones Let 

La misma consulta que antes se puede volver a escribir para usar la función inline (instrucción Let) que recibe `dbName` un parámetro, que se pasa a la función Database ().

```kusto
let foo = (dbName:string)
{
    database(dbName).StormEvents | count
};
foo('help')
```

|Count|
|---|
|59066|

### <a name="use-database-inside-functions"></a>Usar la base de datos () dentro de las funciones 

Se puede volver a escribir la misma consulta anterior para usarla en una función que recibe un parámetro `dbName` , que se pasa a la función Database ().

```kusto
.create function foo(dbName:string)
{
    database(dbName).StormEvents | count
};
```

**Nota:** estas funciones solo se pueden usar de forma local y no en la consulta entre clústeres.

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

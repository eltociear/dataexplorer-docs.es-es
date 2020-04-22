---
title: cluster() (función de ámbito) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe cluster() (función de ámbito) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: c5f537b47006af4035c9db26388c1d4110c69b55
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766112"
---
# <a name="cluster-scope-function"></a>cluster() (función de ámbito)

::: zone pivot="azuredataexplorer"

Cambia la referencia de la consulta a un clúster remoto. 

```kusto
cluster('help').database('Sample').SomeTable
```

**Sintaxis**

`cluster(`*stringConstant*`)`

**Argumentos**

* *stringConstant*: Nombre del clúster al que se hace referencia. El nombre del clúster puede ser un nombre DNS completo `.kusto.windows.net`o una cadena con el sufijo . El argumento tiene que ser _constante_ antes de la ejecución de la consulta, es decir, no puede provenir de la evaluación de la subconsulta.

**Notas**

* Para acceder a la base de datos dentro del mismo clúster- utilice la función [database().](databasefunction.md)
* Más información sobre las consultas entre clústeres y bases de datos disponibles [aquí](cross-cluster-or-database-queries.md)  

## <a name="examples"></a>Ejemplos

### <a name="use-cluster-to-access-remote-cluster"></a>Utilice cluster() para acceder al clúster remoto 

La siguiente consulta se puede ejecutar en cualquiera de los clústeres de Kusto.

```kusto
cluster('help').database('Samples').StormEvents | count

cluster('help.kusto.windows.net').database('Samples').StormEvents | count  
```

|Count|
|---|
|59066|

### <a name="use-cluster-inside-let-statements"></a>Utilice cluster() dentro de las instrucciones let 

La misma consulta que la anterior se puede reescribir para utilizar `clusterName` la función en línea (instrucción let) que recibe un parámetro, que se pasa a la función cluster().

```kusto
let foo = (clusterName:string)
{
    cluster(clusterName).database('Samples').StormEvents | count
};
foo('help')
```

|Count|
|---|
|59066|

### <a name="use-cluster-inside-functions"></a>Usar cluster() dentro de Functions 

La misma consulta que la anterior se puede reescribir para `clusterName` usarla en una función que recibe un parámetro, que se pasa a la función cluster().

```kusto
.create function foo(clusterName:string)
{
    cluster(clusterName).database('Samples').StormEvents | count
};
```

Nota: estas funciones solo se pueden utilizar localmente y no en la consulta entre **clústeres.**

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

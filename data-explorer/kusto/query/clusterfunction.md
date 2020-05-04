---
title: 'Cluster () (función de ámbito): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe cluster () (función SCOPE) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 092915c08b4b3d1e72722a4303e911403b2defd2
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737205"
---
# <a name="cluster-scope-function"></a>Cluster () (función SCOPE)

::: zone pivot="azuredataexplorer"

Cambia la referencia de la consulta a un clúster remoto. 

```kusto
cluster('help').database('Sample').SomeTable
```

**Sintaxis**

`cluster(`*stringConstant*`)`

**Argumentos**

* *stringConstant*: nombre del clúster al que se hace referencia. El nombre del clúster puede ser un nombre DNS completo o una cadena con `.kusto.windows.net`el sufijo. El argumento debe ser _constante_ antes de la ejecución de la consulta, es decir, no puede proviene de la evaluación de la subconsulta.

**Notas**

* Para tener acceso a la base de datos dentro de la misma función de uso de la [base de datos ()](databasefunction.md) de clúster.
* Más información sobre las consultas entre clústeres y entre bases de datos disponibles [aquí](cross-cluster-or-database-queries.md)  

## <a name="examples"></a>Ejemplos

### <a name="use-cluster-to-access-remote-cluster"></a>Usar cluster () para tener acceso al clúster remoto 

La siguiente consulta se puede ejecutar en cualquiera de los clústeres de Kusto.

```kusto
cluster('help').database('Samples').StormEvents | count

cluster('help.kusto.windows.net').database('Samples').StormEvents | count  
```

|Count|
|---|
|59066|

### <a name="use-cluster-inside-let-statements"></a>Usar cluster () dentro de las instrucciones Let 

La misma consulta que antes se puede volver a escribir para usar la función inline (instrucción Let) que recibe `clusterName` un parámetro, que se pasa a la función cluster ().

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

### <a name="use-cluster-inside-functions"></a>Usar cluster () dentro de las funciones 

Se puede volver a escribir la misma consulta anterior para usarla en una función que recibe un parámetro `clusterName` , que se pasa a la función cluster ().

```kusto
.create function foo(clusterName:string)
{
    cluster(clusterName).database('Samples').StormEvents | count
};
```

**Nota:** estas funciones solo se pueden usar de forma local y no en la consulta entre clústeres.

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

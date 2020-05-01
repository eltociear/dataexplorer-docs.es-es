---
title: 'Administración de directivas de Kusto IngestionBatching: Azure Explorador de datos'
description: En este artículo se describe la Directiva IngestionBatching en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: e9823fd0cd44dd2e5bd0731cc59086961ce86d8c
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617772"
---
# <a name="ingestionbatching-policy"></a>Directiva IngestionBatching

La [Directiva ingestionBatching](batchingpolicy.md) es un objeto de directiva que determina cuándo se debe detener la agregación de datos durante la ingesta de datos de acuerdo con la configuración especificada.

La Directiva se puede establecer en `null`, en cuyo caso se usan los valores predeterminados, estableciendo el intervalo máximo de tiempo de procesamiento por lotes en 5 minutos, 1000 elementos y un tamaño total de lote de 1g o el valor de clúster predeterminado establecido por Kusto.

Si no se establece la Directiva para una entidad determinada, buscará una directiva de nivel de jerarquía superior, si todas se establecen en null, se usará el valor predeterminado. 

La Directiva tiene un límite inferior de 10 segundos y no se recomienda usar valores de más de 15 minutos.

## <a name="displaying-the-ingestionbatching-policy"></a>Mostrar la Directiva IngestionBatching

La Directiva se puede establecer en una base de datos o en una tabla y se muestra con uno de los siguientes comandos:

* `.show``database` *DatabaseName* NombreDeBaseDeDatos `policy``ingestionbatching`
* `.show``table` *DatabaseName*NombreDeBaseDeDatos`.`*TableName* TableName `policy``ingestionbatching`

## <a name="altering-the-ingestionbatching-policy"></a>Modificación de la Directiva IngestionBatching

```kusto
.alter <entity_type> <database_or_table_name> policy ingestionbatching @'<ingestionbatching policy json>'
```

Modificar la Directiva IngestionBatching para varias tablas (en el mismo contexto de base de datos):

```kusto
.alter tables (table_name [, ...]) policy ingestionbatching @'<ingestionbatching policy json>'
```

Directiva IngestionBatching:

```kusto
{
  "MaximumBatchingTimeSpan": "00:05:00",
  "MaximumNumberOfItems": 500, 
  "MaximumRawDataSizeMB": 1024
}
```

* `entity_type`: tabla, base de datos
* `database_or_table`: Si la entidad es una tabla o una base de datos, su nombre debe especificarse en el comando como se indica a continuación: 
  - `database_name`, o bien 
  - `database_name.table_name`, o bien 
  - `table_name`(cuando se ejecuta en el contexto de la base de datos específica)

## <a name="deleting-the-ingestionbatching-policy"></a>Eliminación de la Directiva IngestionBatching

```kusto
.delete <entity_type> <database_or_table_name> policy ingestionbatching
```

**Ejemplos**

```kusto
// Show IngestionBatching policy for table `MyTable` in database `MyDatabase`
.show table MyDatabase.MyTable policy ingestionbatching 

// Set IngestionBatching policy on table `MyTable` (in database context) to batch ingress data by 30 seconds, 500 files, or 1GB (whatever comes first)
.alter table MyTable policy ingestionbatching @'{"MaximumBatchingTimeSpan":"00:00:30", "MaximumNumberOfItems": 500, "MaximumRawDataSizeMB": 1024}'

// Set IngestionBatching policy on multiple tables (in database context) to batch ingress data by 1 minute, 20 files, or 300MB (whatever comes first)
.alter tables (MyTable1, MyTable2, MyTable3) policy ingestionbatching @'{"MaximumBatchingTimeSpan":"00:01:00", "MaximumNumberOfItems": 20, "MaximumRawDataSizeMB": 300}'

// Delete IngestionBatching policy on table `MyTable`
.delete table MyTable policy ingestionbatching

// Delete IngestionBatching policy on database `MyDatabase`
.delete database MyDatabase policy ingestionbatching
```

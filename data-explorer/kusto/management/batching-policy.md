---
title: Directiva ingestionBatching - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe la directiva IngestionBatching en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 232767e390669a08312f10d3999d19264fb29f26
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744318"
---
# <a name="ingestionbatching-policy"></a>Política de IngestionBatching

La [directiva ingestionBatching](batchingpolicy.md) es un objeto de directiva que determina cuándo debe detenerse la agregación de datos durante la ingesta de datos según la configuración especificada.

La directiva se `null`puede establecer en , en cuyo caso se utilizan los valores predeterminados, estableciendo el intervalo de tiempo máximo de procesamiento por lotes en: 5 minutos, 1000 elementos y un tamaño de lote total de 1G o el valor de clúster predeterminado establecido por Kusto.

Si la directiva no se establece para una entidad determinada, buscará una directiva de nivel de jerarquía superior, si todas se establecen en null se usará el valor predeterminado. 

La directiva tiene un límite inferior de 10 segundos y no se recomienda utilizar valores mayores de 15 minutos.

## <a name="displaying-the-ingestionbatching-policy"></a>Visualización de la directiva IngestionBatching

La directiva se puede establecer en una base de datos o una tabla y se muestra mediante uno de los siguientes comandos:

* `.show` `database` *DatabaseName* `policy` `ingestionbatching`
* `.show``table` *DatabaseName*`.`*TableName* `policy``ingestionbatching`

## <a name="altering-the-ingestionbatching-policy"></a>Modificación de la política de Ingestión por lotes

```kusto
.alter <entity_type> <database_or_table_name> policy ingestionbatching @'<ingestionbatching policy json>'
```

Modificación de la directiva IngestionBatching para varias tablas (en el mismo contexto de base de datos):

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
* `database_or_table`: si la entidad es tabla o base de datos, su nombre debe especificarse en el comando de la siguiente manera - 
  - `database_name` o 
  - `database_name.table_name` o 
  - `table_name`(cuando se ejecuta en el contexto de la base de datos específica)

## <a name="deleting-the-ingestionbatching-policy"></a>Eliminación de la directiva IngestionBatching

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

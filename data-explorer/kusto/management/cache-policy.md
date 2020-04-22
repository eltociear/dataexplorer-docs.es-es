---
title: 'Directiva de caché: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de caché en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: ca14703b2548bdb23dc3e6e352aeaacbc17303b4
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744470"
---
# <a name="cache-policy"></a>Directiva de caché

En este artículo se describen los comandos utilizados para crear y modificar [la directiva](cachepolicy.md) de caché 

## <a name="displaying-the-cache-policy"></a>Visualización de la directiva de caché

La directiva se puede establecer en una tabla o datos y se muestra mediante uno de los siguientes comandos:

* `.show` `database` *DatabaseName* `policy` `caching`
* `.show``table` *DatabaseName*`.`*TableName* `policy``caching`

## <a name="altering-the-cache-policy"></a>Modificación de la política de caché

```kusto
.alter <entity_type> <database_or_table_name> policy caching hot = <timespan>
```

Modificación de la directiva de caché para varias tablas (en el mismo contexto de base de datos):

```kusto
.alter tables (table_name [, ...]) policy caching hot = <timespan>
```

Política de caché:

```kusto
{
  "DataHotSpan": {
    "Value": "3.00:00:00"
  },
  "IndexHotSpan": {
    "Value": "3.00:00:00"
  }
}
```

* `entity_type`: tabla, base de datos o clúster
* `database_or_table`: si la entidad es tabla o base de datos, su nombre debe especificarse en el comando de la siguiente manera - 
  - `database_name` o 
  - `database_name.table_name` o 
  - `table_name`(cuando se ejecuta en el contexto de la base de datos específica)

## <a name="deleting-the-cache-policy"></a>Eliminación de la directiva de caché

```kusto
.delete <entity_type> <database_or_table_name> policy caching
```

**Ejemplos**

Mostrar directiva de `MyTable` caché `MyDatabase`para la tabla en la base de datos:

```kusto
.show table MyDatabase.MyTable policy caching 
```

Establecer la directiva `MyTable` de caché de la tabla (en el contexto de la base de datos) en 3 días:

```kusto
.alter table MyTable policy caching hot = 3d
```

Establecer la directiva para varias tablas (en contexto de base de datos), en 3 días:

```kusto
.alter tables (MyTable1, MyTable2, MyTable3) policy caching hot = 3d
```

Eliminación de un conjunto de políticas en una tabla:

```kusto
.delete table MyTable policy caching
```

Eliminación de un conjunto de políticas en una base de datos:

```kusto
.delete database MyDatabase policy caching
```

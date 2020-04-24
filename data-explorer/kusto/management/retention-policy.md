---
title: 'Directiva de retención: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de retención en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: b0366bef619d815bbe58f91730eff70ec847c239
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520356"
---
# <a name="retention-policy"></a>Directiva de retención

En este artículo se describen los comandos de control utilizados para crear y modificar la directiva de [retención.](retentionpolicy.md)

## <a name="show-retention-policy"></a>Mostrar política de retención

```kusto
.show <entity_type> <database_or_table> policy retention

.show <entity_type> *  policy retention
```

* `entity_type`: tabla o base de datos
* `database_or_table`: `database_name` `database_name.table_name` o `table_name` (en contexto de base de datos)

**Ejemplo**

Mostrar la directiva de `MyDatabase`retención para la base de datos denominada:

```kusto
.show database MyDatabase policy retention
```

## <a name="delete-retention-policy"></a>Eliminar directiva de retención

La eliminación de la directiva de retención de datos es afectara a la retención ilimitada de datos.

La eliminación de la directiva de retención de datos de la tabla hará que la tabla derive la directiva de retención del nivel de base de datos.

```kusto
.delete <entity_type> <database_or_table> policy retention
```

* `entity_type`: tabla o base de datos
* `database_or_table`: `database_name` `database_name.table_name` o `table_name` (en contexto de base de datos)

**Ejemplo**

Elimine la directiva de `MyTable1`retención de la tabla denominada :

```kusto
.delete table MyTable policy retention
```


## <a name="alter-retention-policy"></a>Modificar la política de retención

```kusto
.alter <entity_type> <database_or_table> policy retention <retention_policy>

.alter tables (<table_name> [, ...]) policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table> policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table_name> policy retention [softdelete = <timespan>] [recoverability = disabled|enabled]
```

* `entity_type`: tabla o base de datos
* `database_or_table`: `database_name` `database_name.table_name` o `table_name` (en contexto de base de datos)
* `table_name`: nombre de una tabla en un contexto de base de datos.  Un comodín (se`*` permite aquí).
* `retention_policy` :

```
    "{ 
        \"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Disabled\"
    }" 
```

**Ejemplos**

Mostrar la directiva de `MyDatabase`retención para la base de datos denominada:

```kusto
.show database MyDatabase policy retention
```

Establece una directiva de retención con un período de eliminación temporal de 10 días y capacidad de recuperación de datos deshabilitada:

```kusto
.alter-merge table Table1 policy retention softdelete = 10d recoverability = disabled
```

Establece una directiva de retención con un período de eliminación temporal de 10 días y capacidad de recuperación de datos habilitada:

```kusto
.alter table Table1 policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

Establece la misma directiva de retención que la anterior, pero esta vez para varias tablas (Table1, Table2 y Table3):

```kusto
.alter tables (Table1, Table2, Table3) policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

Establece una directiva de retención con los valores predeterminados: 100 años como período de eliminación temporal y capacidad de recuperación habilitada:

```kusto
.alter table Table1 policy retention "{}"
```
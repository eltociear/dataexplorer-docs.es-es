---
title: 'Administración de directivas de retención de Kusto: Azure Explorador de datos'
description: En este artículo se describe la Directiva de retención en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: e03e529e0c802f0d424deb4048c5809bbe845ddd
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617415"
---
# <a name="retention-policy"></a>Directiva de retención

En este artículo se describen los comandos de control que se usan para crear y modificar [directivas de retención](retentionpolicy.md).

## <a name="show-retention-policy"></a>Mostrar Directiva de retención

```kusto
.show <entity_type> <database_or_table> policy retention

.show <entity_type> *  policy retention
```

* `entity_type`: tabla o base de datos
* `database_or_table`: `database_name` o `database_name.table_name` o `table_name` (en el contexto de la base de datos)

**Ejemplo**

Mostrar la Directiva de retención para la base `MyDatabase`de datos denominada:

```kusto
.show database MyDatabase policy retention
```

## <a name="delete-retention-policy"></a>Eliminar Directiva de retención

La eliminación de la Directiva de retención de datos es affectively.

La eliminación de la Directiva de retención de datos de la tabla hará que la tabla derive la Directiva de retención del nivel de base de datos.

```kusto
.delete <entity_type> <database_or_table> policy retention
```

* `entity_type`: tabla o base de datos
* `database_or_table`: `database_name` o `database_name.table_name` o `table_name` (en el contexto de la base de datos)

**Ejemplo**

Elimine la Directiva de retención de la `MyTable1`tabla denominada:

```kusto
.delete table MyTable policy retention
```


## <a name="alter-retention-policy"></a>Modificar Directiva de retención

```kusto
.alter <entity_type> <database_or_table> policy retention <retention_policy>

.alter tables (<table_name> [, ...]) policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table> policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table_name> policy retention [softdelete = <timespan>] [recoverability = disabled|enabled]
```

* `entity_type`: tabla o base de datos
* `database_or_table`: `database_name` o `database_name.table_name` o `table_name` (en el contexto de la base de datos)
* `table_name`: nombre de una tabla en un contexto de base de datos.  Un carácter comodín`*` (se permite aquí).
* `retention_policy` :

```kusto
    "{ 
        \"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Disabled\"
    }" 
```

**Ejemplos**

Mostrar la Directiva de retención para la base `MyDatabase`de datos denominada:

```kusto
.show database MyDatabase policy retention
```

Establece una directiva de retención con un período de eliminación temporal de 10 días y una capacidad de recuperación de datos deshabilitada:

```kusto
.alter-merge table Table1 policy retention softdelete = 10d recoverability = disabled
```

Establece una directiva de retención con un período de eliminación temporal de 10 días y la capacidad de recuperación de datos habilitada:

```kusto
.alter table Table1 policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

Establece la misma directiva de retención anterior, pero esta vez para varias tablas (Table1, Tabla2 y Table3):

```kusto
.alter tables (Table1, Table2, Table3) policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

Establece una directiva de retención con los valores predeterminados: 100 años como el período de eliminación temporal y la capacidad de recuperación habilitada:

```kusto
.alter table Table1 policy retention "{}"
```
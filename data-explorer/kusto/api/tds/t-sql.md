---
title: T-SQL-Azure Explorador de datos
description: En este artículo se describe T-SQL en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1115414358a1025d4931484b81d6eda76109e6cd
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373525"
---
# <a name="t-sql-support"></a>Compatibilidad con T-SQL

El [lenguaje de consulta de Kusto (KQL)](../../query/index.md) es el lenguaje de consulta preferido.
Sin embargo, la compatibilidad con T-SQL es útil para las herramientas que no se pueden convertir fácilmente para usar KQL.  
La compatibilidad con T-SQL también es útil para el uso ocasional de personas familiarizadas con SQL.

Kusto puede interpretar y ejecutar consultas de T-SQL con algunas limitaciones de lenguaje.

> [!NOTE]
> Kusto no admite comandos DDL. Solo `select` se admiten instrucciones T-SQL. Para obtener más información sobre las principales diferencias con respecto a T-SQL, consulte [problemas conocidos de SQL](./sqlknownissues.md).

## <a name="querying-from-kustoexplorer-with-t-sql"></a>Consulta desde Kusto. Explorer con T-SQL

La herramienta Kusto. Explorer admite consultas de T-SQL a Kusto.
Para indicar a Kusto. Explorer que ejecute una consulta, comience la consulta con una línea de comentario T-SQL vacía ( `--` ). Por ejemplo:

```sql
--
select * from StormEvents
```

## <a name="from-t-sql-to-kusto-query-language"></a>Del lenguaje de consulta de T-SQL a Kusto

Kusto admite la traducción de consultas de T-SQL al lenguaje de consulta de Kusto (KQL). Esta traducción puede ayudar a los usuarios familiarizados con SQL para comprender mejor a KQL.
Para recuperar la KQL equivalente desde alguna instrucción T-SQL `select` , agregue `explain` antes de la consulta.

Por ejemplo, la siguiente consulta de T-SQL:

```sql
--
explain
select top(10) *
from StormEvents
order by DamageProperty desc
```

genera este resultado:

```kusto
StormEvents
| sort by DamageProperty desc nulls first
| take 10
```

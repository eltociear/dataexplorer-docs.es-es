---
title: T-SQL - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe T-SQL en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d262d8b7587296c02a2a31d850919af0931bcde6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523416"
---
# <a name="t-sql"></a>T-SQL

El servicio Kusto puede interpretar y ejecutar consultas T-SQL con algunas limitaciones de idioma.
Aunque el lenguaje de [consulta Kusto](../../query/index.md) es el idioma preferido para Kusto, esta compatibilidad es útil para la herramienta existente que no se puede convertir fácilmente para usar el lenguaje de consulta preferido y para el uso ocasional de Kusto por personas familiares con SQL.

> [!NOTE]
> Kusto no admite un comando DDL de esta `SELECT` manera, solo se admiten instrucciones T-SQL. Consulte [Problemas conocidos](./sqlknownissues.md) de SQL para obtener más información sobre las principales diferencias entre SQL Server y Kusto con respecto a T-SQL.

## <a name="querying-kusto-from-kustoexplorer-with-t-sql"></a>Consulta de Kusto desde Kusto.Explorer con T-SQL

La herramienta Kusto.Explorer admite el envío de consultas T-SQL a Kusto.
Para indicar a Kusto.Explorer que ejecute una consulta en este modo, anteponga a la consulta una línea de comentario T-SQL vacía. Por ejemplo:

```sql
--
select * from StormEvents
```

## <a name="from-t-sql-to-kusto-query-language"></a>Del lenguaje de consulta de T-SQL a Kusto

Kusto admite la traducción de consultas T-SQL al lenguaje de consulta Kusto. Esto lo pueden usar, por ejemplo, personas familiares con SQL que quieran comprender mejor el lenguaje de consulta Kusto. Para recuperar el lenguaje de consulta Kusto `SELECT` equivalente a `EXPLAIN` alguna instrucción T-SQL, simplemente agregue antes de la consulta.

Por ejemplo, la siguiente consulta T-SQL:

```sql
--
explain
select top(10) *
from StormEvents
order by DamageProperty desc
```

Produce esta salida:

```kusto
StormEvents
| sort by DamageProperty desc nulls first
| take 10
```
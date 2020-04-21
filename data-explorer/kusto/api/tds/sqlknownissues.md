---
title: Diferencias de MS-TDS/T-SQL entre Kusto Microsoft SQL Server - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describen las diferencias de MS-TDS/T-SQL entre Kusto Microsoft SQL Server en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/04/2019
ms.openlocfilehash: be294053fdd0f95d488f52b547ef7abd0ef7c23c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523433"
---
# <a name="ms-tdst-sql-differences-between-kusto-microsoft-sql-server"></a>Diferencias de MS-TDS/T-SQL entre Kusto Microsoft SQL Server

A continuación se muestra una lista parcial de las principales diferencias entre la implementación de T-SQL de Kusto y SQL Server.

## <a name="create-insert-drop-alter-statements"></a>Instrucciones CREATE, INSERT, DROP, ALTER

Kusto no admite modificaciones de esquema ni modificaciones de datos a través de MS-TDS, ni admite las instrucciones T-SQL anteriores.

## <a name="correlated-sub-queries"></a>Subconsultas correlacionadas

Kusto no admite subconsultas correlacionadas `SELECT` `WHERE`en `JOIN` cláusulas , y .

## <a name="top-flavors"></a>Sabores TOP

Kusto omite `WITH TIES` y evalúa la `TOP`consulta como regular.
Kusto no es `PERCENT`compatible con .

## <a name="cursors"></a>Cursores

Kusto no admite cursores SQL.

## <a name="flow-control"></a>Control de flujo

Kusto no admite instrucciones de control de flujo, excepto `IF` `THEN` `ELSE` en algunos casos limitados, como la cláusula que tiene el esquema idéntico para los `THEN` lotes y. `ELSE`

## <a name="data-types"></a>Tipos de datos

Según la consulta, los datos devueltos pueden tener un tipo diferente que en SQL Server.
Un ejemplo obvio aquí `TINYINT` son `SMALLINT` tipos como y que no tienen equivalente en Kusto. Por lo tanto, los `BYTE` clientes que esperan un valor de tipo o `INT16` pueden obtener un `INT32` valor o un `INT64` valor en su lugar.

## <a name="column-order-in-results"></a>Orden de columna en los resultados

Cuando se usa asterix en la `SELECT` instrucción, el orden de las columnas de cada conjunto de resultados puede diferir entre Kusto y SQL Server. El cliente que usa nombres de columna funcionaría mejor en estos casos.
Si no hay ningún carácter `SELECT` asterix en la instrucción, se conservarán los ordinales de columna.

## <a name="columns-name-in-results"></a>Nombre de las columnas en los resultados

En T-SQL, varias columnas pueden tener el mismo nombre. Esto no está permitido en Kusto.
En caso de una colisión en los nombres, los nombres de las columnas pueden ser diferentes en Kusto.
Sin embargo, el nombre original se conservaría, al menos para una de las columnas.

## <a name="any-all-and-exists-predicates"></a>ANY, ALL y EXISTS predicados

Kusto no admite los `ANY`predicados , `ALL`, y `EXISTS`.

## <a name="recursive-ctes"></a>CTE recursivas

Kusto no admite expresiones de tabla comunes recursivas.

## <a name="dynamic-sql"></a>SQL dinámico

Kusto no admite instrucciones SQL dinámicas (ejecución en línea del script SQL generado por la consulta).

## <a name="within-group"></a>WITHIN GROUP

Kusto no admite `WITHIN GROUP` cláusulas.

## <a name="truncate-function"></a>Función TRUNCATE

La función TRUNCATE (ODBC) en Kusto funciona de forma similar a ROUND, lo que significa que el resultado será el valor más cercano en lugar del inferior devuelto en SQL.
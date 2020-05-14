---
title: Diferencias entre Kusto MS-TDS/T-SQL con SQL Server Azure Explorador de datos
description: En este artículo se describen las diferencias de MS-TDS/T-SQL entre Kusto Microsoft SQL Server en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/04/2019
ms.openlocfilehash: c949b5bb3659d82ed586a39b4310495e61934777
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382376"
---
# <a name="ms-tdst-sql-differences-between-kusto-microsoft-sql-server"></a>Diferencias entre MS-TDS y T-SQL entre Kusto Microsoft SQL Server

A continuación se muestra una lista parcial de las principales diferencias entre Kusto SQL Server y la implementación de T-SQL.

## <a name="create-insert-drop-alter-statements"></a>Instrucciones CREATE, INSERT, DROP, ALTER

Kusto no admite modificaciones de esquema o modificaciones de datos a través de MS-TDS, ni tampoco admite las instrucciones T-SQL anteriores.

## <a name="correlated-sub-queries"></a>Subconsultas correlacionadas

Kusto no admite las subconsultas correlacionadas en las `SELECT` `WHERE` `JOIN` cláusulas, y.

## <a name="top-flavors"></a>Versiones principales

Kusto omite `WITH TIES` y evalúa la consulta como normal `TOP` .
Kusto no admite `PERCENT` .

## <a name="cursors"></a>Cursores

Kusto no admite cursores de SQL.

## <a name="flow-control"></a>Control de flujo

Kusto no es compatible con las instrucciones de control de flujo, excepto en algunos casos limitados, como la `IF` `THEN` `ELSE` cláusula que tiene el esquema idéntico para los `THEN` `ELSE` lotes y.

## <a name="data-types"></a>Tipos de datos

Dependiendo de la consulta, los datos devueltos pueden tener un tipo diferente que en SQL Server.
Un ejemplo obvio aquí son tipos como `TINYINT` y `SMALLINT` que no tienen ningún equivalente en Kusto. Por lo tanto, los clientes que esperan un valor de tipo `BYTE` o `INT16` pueden obtener un `INT32` `INT64` valor o en su lugar.

## <a name="column-order-in-results"></a>Orden de las columnas en los resultados

Cuando se usa asterisco en la `SELECT` instrucción, el orden de las columnas de cada conjunto de resultados puede diferir entre Kusto y SQL Server. El cliente que usa nombres de columna funcionaría mejor en estos casos.
Si no hay ningún carácter asterisco en la `SELECT` instrucción, se conservarán los ordinales de columna.

## <a name="columns-name-in-results"></a>Nombre de las columnas en los resultados

En T-SQL, varias columnas pueden tener el mismo nombre. Esto no está permitido en Kusto.
En el caso de una colisión en los nombres, los nombres de las columnas pueden ser diferentes en Kusto.
Sin embargo, se conservaría el nombre original, al menos para una de las columnas.

## <a name="any-all-and-exists-predicates"></a>Predicados ANY, ALL y EXISTs

Kusto no admite los predicados `ANY` , `ALL` y `EXISTS` .

## <a name="recursive-ctes"></a>CTE recursivas

Kusto no es compatible con las expresiones de tabla comunes recursivas.

## <a name="dynamic-sql"></a>SQL dinámico

Kusto no admite instrucciones SQL dinámicas (ejecución en línea del script SQL generado por la consulta).

## <a name="within-group"></a>WITHIN GROUP

Kusto no admite la `WITHIN GROUP` cláusula.

## <a name="truncate-function"></a>TRUNCAte (función)

TRUNCAte function (ODBC) en Kusto funciona de forma similar a ROUND, lo que significa que el resultado será el valor más cercano en lugar del más bajo devuelto en SQL.
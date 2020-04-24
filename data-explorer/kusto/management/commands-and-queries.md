---
title: 'Administración de comandos y consultas: Explorador de Azure Data Explorer Microsoft Docs'
description: En este artículo se describe la administración de comandos y consultas en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/19/2019
ms.openlocfilehash: f8c01709d69a0b439ce51b4782eb8f747db15d65
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521920"
---
# <a name="commands-and-queries-management"></a>Gestión de comandos y consultas

## <a name="show-commands-and-queries"></a>.show comandos y consultas 

`.show``commands-and-queries` devuelve una tabla con comandos admin y consultas que han alcanzado un estado final. Estos comandos y consultas están disponibles para consultar durante 30 días.

La información presentada en la salida del comando es similar a la presentada por los [comandos .show](commands.md) y las [consultas .show,](queries.md)sin embargo, esencialmente permite unir ambos conjuntos de resultados de una manera sencilla.

**Sintaxis**

`.show` `commands-and-queries`
 
**Salida**
 
El esquema de salida es el siguiente:

| ColumnName               | ColumnType |
|--------------------------|------------|
| ClientActivityId         | string     |
| CommandType              | string     |
| Texto                     | string     |
| Base de datos                 | string     |
| StartedOn                | datetime   |
| LastUpdatedOn            | datetime   |
| Duration                 | timespan   |
| State                    | string     |
| FailureReason            | string     |
| RootActivityId           | guid       |
| Usuario                     | string     |
| Application              | string     |
| Principal                | string     |
| ClientRequestProperties  | dinámico    |
| TotalCpu                 | timespan   |
| MemoryPeak               | long       |
| CacheStatistics          | dinámico    |
| ScannedExtentsStatistics | dinámico    |
| ResultSetStatistics      | dinámico    |

Tenga en cuenta que para `CommandType` `Query`las consultas, el valor de es .
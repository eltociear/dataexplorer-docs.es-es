---
title: 'Administración de comandos y consultas: Azure Explorador de datos'
description: En este artículo se describe la administración de comandos y consultas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/19/2019
ms.openlocfilehash: c7f692739071496ce492d168c6036a2c2adac8fd
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84329053"
---
# <a name="commands-and-queries-management"></a>Administración de comandos y consultas

## <a name="show-commands-and-queries"></a>. Mostrar comandos y consultas 

`.show``commands-and-queries`devuelve una tabla con comandos de administrador y consultas que han alcanzado un estado final. Estos comandos y consultas están disponibles durante 30 días.

La información que se presenta en la salida del comando es similar a [. Mostrar comandos](commands.md) y [. Mostrar consultas](queries.md), pero básicamente le permite combinar ambos conjuntos de resultados de una manera sencilla.

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
| Inicio de                | datetime   |
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

> [!NOTE]
> En el caso de las consultas, el valor de `CommandType` es `Query` .

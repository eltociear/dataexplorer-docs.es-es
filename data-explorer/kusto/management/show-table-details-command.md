---
title: '. Mostrar detalles de la tabla: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Mostrar los detalles de la tabla en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 3182c059a3f56b94950009672759388bbff8058d
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616922"
---
# <a name="show-table-details"></a>.show table details
Devuelve un conjunto que contiene la tabla especificada o todas las tablas de la base de datos con un resumen detallado de las propiedades de cada tabla.

Requiere el [permiso visor de bases de datos](../management/access-control/role-based-authorization.md).

```kusto
.show table T1 details
.show tables (T1, ..., Tn) details
.show tables details
```

**Salida**

| Parámetro de salida           | Tipo     | Descripción                                                                                     |
|----------------------------|----------|-------------------------------------------------------------------------------------------------|
| `TableName`                | String   | Nombre de la tabla.                                                                          |
| `DatabaseName`             | String   | La base de datos a la que pertenece la tabla.                                                         |
| `Folder`                   | String   | La carpeta de la tabla.                                                                             |
| `DocString`                | String   | Cadena que documenta la tabla.                                                                 |
| `TotalExtents`             | Int64    | El número total de extensiones en la tabla.                                                       |
| `TotalExtentSize`          | Double   | Tamaño total de las extensiones (tamaño comprimido + tamaño del índice) en la tabla.                          |
| `TotalOriginalSize`        | Double   | Tamaño original total de los datos de la tabla.                                                   |
| `TotalRowCount`            | Int64    | Número total de filas de la tabla.                                                          |
| `HotExtents`               | Int64    | El número total de extensiones en la tabla, almacenadas en la memoria caché activa.                              |
| `HotExtentSize`            | Double   | El tamaño total de las extensiones (tamaño comprimido + tamaño del índice) en la tabla, almacenados en la memoria caché activa. |
| `HotOriginalSize`          | Double   | Tamaño original total de los datos de la tabla, almacenados en la memoria caché activa.                          |
| `HotRowCount`              | Int64    | El número total de filas de la tabla, almacenadas en la memoria caché activa.                                 |
| `AuthorizedPrincipals`     | String   | Entidades de seguridad autorizadas de la tabla, serializadas como JSON.                                          |
| `RetentionPolicy`          | String   | La Directiva de retención`*` efectiva de la tabla, serializada como JSON.                                  |
| `CachingPolicy`            | String   | La Directiva de almacenamiento`*` en caché efectiva de la tabla, serializada como JSON.                                    |
| `ShardingPolicy`           | String   | La Directiva de particionamiento efectiva`*` de la tabla, serializada como JSON. 66666666666666                     |
| `MergePolicy`              | String   | La Directiva de combinación`*` efectiva de la tabla, serializada como JSON.                                      |
| `StreamingIngestionPolicy` | String   | La Directiva de ingesta de streaming efectiva`*` de la tabla, serializada como JSON.                        |
| `MinExtentsCreationTime`   | DateTime | El tiempo de creación mínimo de una extensión en la tabla (o null, si no hay ninguna extensión).         |
| `MaxExtentsCreationTime`   | DateTime | El tiempo de creación máximo de una extensión en la tabla (o null, si no hay ninguna extensión).         |
| `RowOrderPolicy`           | String   | La Directiva de orden de filas efectiva de la tabla, serializada como JSON.                                     |

`*`*Teniendo en cuenta las directivas de entidades primarias (como base de datos o clúster).*

**Ejemplo de salida**

| TableName         | DatabaseName | Carpeta | DocString | TotalExtents | TotalExtentSize | TotalOriginalSize | TotalRowCount | HotExtents | HotExtentSize | HotOriginalSize | HotRowCount | AuthorizedPrincipals                                                                                                                                                                               | RetentionPolicy                                                                                                                                       | CachingPolicy                                                                        | ShardingPolicy                                                                    | MergePolicy                                                                                                                                             | StreamingIngestionPolicy | MinExtentsCreationTime      | MaxExtentsCreationTime      |
|-------------------|--------------|--------|-----------|--------------|-----------------|-------------------|---------------|------------|---------------|-----------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-----------------------------|-----------------------------|
| Operaciones        | Operaciones   |        |           | 1164         | 37687203        | 53451358          | 223325        | 29         | 838752        | 1388213         | 5117        | [{"Type": "AAD user", "DisplayName": "My Name (UPN: alias@fabrikam.com)", "objectId": "a7a77777-4c21-4649-95c5-350bf486087b", "FQN": "aaduser = a7a77777-4c21-4649-95c5-350bf486087b", "Notes": ""}] | {"SoftDeletePeriod": "365.00:00:00", "ContainerRecyclingPeriod": "1.00:00:00", "ExtentsDataSizeLimitInBytes": 0, "OriginalDataSizeLimitInBytes": 0}  | {"DataHotSpan": "4.00:00:00", "IndexHotSpan": "4.00:00:00", "ColumnOverrides": []} | {"MaxRowCount": 750000, "MaxExtentSizeInMb": 1024, "MaxOriginalSizeInMb": 2048} | {"RowCountUpperBoundForMerge": 0, "MaxExtentsToMerge": 100, "LoopPeriod": "01:00:00", "MaxRangeInHours": 3, "AllowRebuild": true, "AllowMerge": true} | null                     |
| ServiceOperations | Operaciones   |        |           | 1109         | 76588803        | 91553069          | 110125        | 27         | 2635742       | 2929926         | 3162        | [{"Type": "AAD user", "DisplayName": "My Name (UPN: alias@fabrikam.com)", "objectId": "a7a77777-4c21-4649-95c5-350bf486087b", "FQN": "aaduser = a7a77777-4c21-4649-95c5-350bf486087b", "Notes": ""}] | {"SoftDeletePeriod": "365.00:00:00", "ContainerRecyclingPeriod": "1.00:00:00", "ExtentsDataSizeLimitInBytes": 0, "OriginalDataSizeLimitInBytes": 0} | {"DataHotSpan": "4.00:00:00", "IndexHotSpan": "4.00:00:00", "ColumnOverrides": []} | {"MaxRowCount": 750000, "MaxExtentSizeInMb": 1024, "MaxOriginalSizeInMb": 2048} | {"RowCountUpperBoundForMerge": 0, "MaxExtentsToMerge": 100, "LoopPeriod": "01:00:00", "MaxRangeInHours": 3, "AllowRebuild": true, "AllowMerge": true} | null                     | 2018-02-08 15:30:38.8489786 | 2018-02-14 07:47:28.7660267 |

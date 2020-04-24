---
title: 'Detalles de la tabla .show: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los detalles de la tabla .show en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 6197e173df417acb1f5dc506337773f9534564bf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519795"
---
# <a name="show-table-details"></a>Detalles de la tabla .show
Devuelve un conjunto que contiene la tabla especificada o todas las tablas de la base de datos con un resumen detallado de las propiedades de cada tabla.

Requiere [permiso Visor](../management/access-control/role-based-authorization.md)de base de datos .

```
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
| `DocString`                | String   | Una cadena que documenta la tabla.                                                                 |
| `TotalExtents`             | Int64    | El número total de extensiones de la tabla.                                                       |
| `TotalExtentSize`          | Double   | El tamaño total de las extensiones (tamaño comprimido + tamaño de índice) en la tabla.                          |
| `TotalOriginalSize`        | Double   | El tamaño original total de los datos de la tabla.                                                   |
| `TotalRowCount`            | Int64    | El número total de filas de la tabla.                                                          |
| `HotExtents`               | Int64    | El número total de extensiones de la tabla, almacenadas en la caché activa.                              |
| `HotExtentSize`            | Double   | El tamaño total de las extensiones (tamaño comprimido + tamaño de índice) de la tabla, almacenada en la caché activa. |
| `HotOriginalSize`          | Double   | El tamaño original total de los datos de la tabla, almacenados en la caché activa.                          |
| `HotRowCount`              | Int64    | El número total de filas de la tabla, almacenadas en la caché activa.                                 |
| `AuthorizedPrincipals`     | String   | Las entidades de seguridad autorizadas de la tabla, serializadas como JSON.                                          |
| `RetentionPolicy`          | String   | La política de`*` retención efectiva de la tabla, serializada como JSON.                                  |
| `CachingPolicy`            | String   | La política de`*` almacenamiento en caché efectiva de la tabla, serializada como JSON.                                    |
| `ShardingPolicy`           | String   | Política de partición efectiva`*` de la tabla, serializada como JSON.6666666666666666666                     |
| `MergePolicy`              | String   | La política de`*` combinación efectiva de la tabla, serializada como JSON.                                      |
| `StreamingIngestionPolicy` | String   | La eficaz política`*` de ingesta de streaming de la tabla, serializada como JSON.                        |
| `MinExtentsCreationTime`   | DateTime | El tiempo mínimo de creación de una extensión en la tabla (o null, si no hay extensiones).         |
| `MaxExtentsCreationTime`   | DateTime | El tiempo máximo de creación de una extensión en la tabla (o null, si no hay extensiones).         |
| `RowOrderPolicy`           | String   | La política de orden de fila efectiva de la tabla, serializada como JSON.                                     |

`*`*Teniendo en cuenta las directivas de las entidades principales (como la base de datos/clúster).*

**Ejemplo de salida**

| TableName         | DatabaseName | Carpeta | DocString | TotalExtents | TotalExtentSize | TotalOriginalSize | TotalRowCount | HotExtents | HotExtentSize | HotOriginalSize | HotRowCount | AuthorizedPrincipals                                                                                                                                                                               | RetentionPolicy                                                                                                                                       | CachingPolicy                                                                        | ShardingPolicy                                                                    | MergePolicy                                                                                                                                             | StreamingIngestionPolicy | MinExtentsCreationTime      | MaxExtentsCreationTime      |
|-------------------|--------------|--------|-----------|--------------|-----------------|-------------------|---------------|------------|---------------|-----------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-----------------------------|-----------------------------|
| Operaciones        | Operaciones   |        |           | 1164         | 37687203        | 53451358          | 223325        | 29         | 838752        | 1388213         | 5117        | ["Tipo": "Usuario de AAD", "DisplayName": "Mi alias@fabrikam.comnombre (upn: )", "ObjectId": "a7a77777-4c21-4649-95c5-350bf486087b", "FQN": "aaduser-a7a7777-4c21-4649-95c5-35bf0bf086087b", "Notes": """ | "SoftDeletePeriod": "365.00:00:00", "ContainerRecyclingPeriod": "1.00:00:00", "ExtentsDataSizeLimitInBytes": 0, "OriginalDataSizeLimitInBytes": 0 ?  | • "DataHotSpan": "4.00:00:00", "IndexHotSpan": "4.00:00:00", "ColumnOverrides": [] | "MaxRowCount": 750000, "MaxExtentSizeInMb": 1024, "MaxOriginalSizeInMb": 2048 ? | • "RowCountUpperBoundForMerge": 0, "MaxExtentsToMerge": 100, "LoopPeriod": "01:00:00", "MaxRangeInHours": 3, "AllowRebuild": true, "AllowMerge": true ? | null                     |
| ServiceOperations | Operaciones   |        |           | 1109         | 76588803        | 91553069          | 110125        | 27         | 2635742       | 2929926         | 3162        | ["Tipo": "Usuario de AAD", "DisplayName": "Mi alias@fabrikam.comnombre (upn: )", "ObjectId": "a7a77777-4c21-4649-95c5-350bf486087b", "FQN": "aaduser-a7a7777-4c21-4649-95c5-35bf0bf086087b", "Notes": """ | • "SoftDeletePeriod": "365.00:00:00", "ContainerRecyclingPeriod": "1.00:00:00", "ExtentsDataSizeLimitInBytes": 0, "OriginalDataSizeLimitInBytes": 0 ? | • "DataHotSpan": "4.00:00:00", "IndexHotSpan": "4.00:00:00", "ColumnOverrides": [] | "MaxRowCount": 750000, "MaxExtentSizeInMb": 1024, "MaxOriginalSizeInMb": 2048 ? | • "RowCountUpperBoundForMerge": 0, "MaxExtentsToMerge": 100, "LoopPeriod": "01:00:00", "MaxRangeInHours": 3, "AllowRebuild": true, "AllowMerge": true ? | null                     | 2018-02-08 15:30:38.8489786 | 2018-02-14 07:47:28.7660267 |
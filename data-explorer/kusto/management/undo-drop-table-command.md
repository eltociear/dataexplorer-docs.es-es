---
title: Tabla de colocación .undo- Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describe la tabla de colocación .undo en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 55fd08ce2624c522b971e1ee3862f7d11c6f679f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519557"
---
# <a name="undo-drop-table"></a>.undo drop table

`.undo` `drop` El `table` comando revierte una operación de tabla de colocación a una versión de base de datos específica.

**Sintaxis**

`.undo``drop` *TableName* `as` `version=v` *DB_MajorVersion.DB_MinorVersion* *NewTableName*NombreTabla [ NewTableName ] DB_MajorVersion.DB_MinorVersion `table`

El comando debe ejecutarse con el contexto de la base de datos.

**Devuelve**

Este comando:
* Devuelve la lista de extensiones de tabla originales
* Especifica para cada extensión el número de registros que contiene la extensión
* Devuelve si la operación de recuperación se realizó correctamente o no se produjo un error
* Devuelve el motivo del error, si procede.

| ExtentId                             | NumberOfRecords | Situación                   | FailureReason                                                                                                                  |
|--------------------------------------|-----------------|--------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| ef296c9e-d75d-44bc-985c-b93dd2519691 | 100             | Recuperado                |
| 370b30d7-cf2a-4997-986e-3d05f49c9689 | 1000            | Recuperado                |
| 861f18a5-6cde-4f1e-a003-a43506f9e8da | 855             | Incapaz de recuperar la extensión | Contenedor de extensión: 4b47fd84-c7db-4cfb-9378-67c1de7bf154 no se encontró, la extensión se quitó del almacenamiento y no se puede restaurar |

**Ejemplos**

```
// Recover TestTable table to database version 24.3
.undo drop table TestTable version="v24.3"
```

```
// Recover TestTable table to database version 10.3 with new table name, NewTestTable (can be used if a table with the same name was already created since the drop)  
.undo drop table TestTable as NewTestTable version="v10.3"
```

**Cómo encontrar la versión de base de datos requerida**

Puede encontrar la versión de la base `.show` `journal` de datos antes de que se ejecutara la operación de colocación mediante el comando :

```
.show database TestDB journal | where Event == "DROP-TABLE" and EntityName == "TestTable" | project OriginalEntityVersion 
```

| OriginalEntityVersion |
|-----------------------|
| v24.3                 |

**Limitaciones**

Si se ejecutó un comando Purgar en esta base de datos, el comando deshacer tabla de colocación no se puede ejecutar en una versión anterior a la ejecución de purga.

La extensión solo se puede recuperar si aún no se ha alcanzado el período de eliminación de la extensión en la que reside.

El comando requiere el permiso de administrador de base de [datos.](../management/access-control/role-based-authorization.md)
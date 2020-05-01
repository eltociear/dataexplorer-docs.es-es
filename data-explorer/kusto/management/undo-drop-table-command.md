---
title: '. Undo DROP TABLE: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Undo DROP TABLE en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 2682afaa9e589a8787b7e42a83e3bb68a24a2781
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616684"
---
# <a name="undo-drop-table"></a>.undo drop table

`.undo` `drop` El `table` comando revierte una operación de eliminación de tabla a una versión de base de datos específica.

**Sintaxis**

`.undo``drop` `as` *NewTableName* *TableName* `version=v` *DB_MajorVersion.DB_MinorVersion* TableName [NewTableName] DB_MajorVersion. DB_MinorVersion `table`

El comando debe ejecutarse con el contexto de la base de datos.

**Devuelve**

Este comando:
* Devuelve la lista de extensiones de tabla originales
* Especifica para cada extensión el número de registros que contiene la extensión
* Devuelve si la operación de recuperación se realizó correctamente o no.
* Devuelve la razón del error, si procede.

| ExtentId                             | NumberOfRecords | Status                   | FailureReason                                                                                                                  |
|--------------------------------------|-----------------|--------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| ef296c9e-d75d-44bc-985c-b93dd2519691 | 100             | Recuperar                |
| 370b30d7-cf2a-4997-986e-3d05f49c9689 | 1000            | Recuperar                |
| 861f18a5-6cde-4f1e-a003-a43506f9e8da | 855             | No se puede recuperar la extensión | Contenedor de extensión: no se encontró 4b47fd84-c7db-4cfb-9378-67c1de7bf154, la extensión se quitó del almacenamiento y no se puede restaurar |

**Ejemplos**

```kusto
// Recover TestTable table to database version 24.3
.undo drop table TestTable version="v24.3"
```

```kusto
// Recover TestTable table to database version 10.3 with new table name, NewTestTable (can be used if a table with the same name was already created since the drop)  
.undo drop table TestTable as NewTestTable version="v10.3"
```

**Cómo buscar la versión necesaria de la base de datos**

Puede buscar la versión de la base de datos antes de ejecutar la operación de `.show` `journal` eliminación mediante el comando:

```kusto
.show database TestDB journal | where Event == "DROP-TABLE" and EntityName == "TestTable" | project OriginalEntityVersion 
```

| OriginalEntityVersion |
|-----------------------|
| v 24.3                 |

**Limitaciones**

Si se ha ejecutado un comando de purga en esta base de datos, el comando Deshacer quitar tabla no se puede ejecutar en una versión anterior a la ejecución de purga.

La extensión solo se puede recuperar si aún no se ha alcanzado el período de eliminación de hardware del contenedor de extensión en el que reside.

El comando requiere [permisos de administrador de base de datos](../management/access-control/role-based-authorization.md).

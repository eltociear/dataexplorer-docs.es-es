---
title: 'Administración de tablas: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la administración de tablas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/27/2020
ms.openlocfilehash: ed70e01f7d955ba92806e7e11f740490e87cc664
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011455"
---
# <a name="tables-management"></a>Administración de tablas

En este tema se describe el ciclo de vida de las tablas y los comandos de control asociados.

Seleccione los vínculos de la tabla siguiente para obtener más información sobre ellos.

| Comandos                                                                                                                 | Operación                       |
|--------------------------------------------------------------------------------------------------------------------------|---------------------------------|
| [`.alter table docstring`](alter-table-docstring-command.md), [`.alter table folder`](alter-table-folder-command.md)                                                                                                                                                                                                   | Administrar propiedades de presentación de tabla |
| [`.create ingestion mapping`](create-ingestion-mapping-command.md), [`.show ingestion mappings`](show-ingestion-mapping-command.md), [`.alter ingestion mapping`](alter-ingestion-mapping-command.md), [`.drop ingestion mapping`](drop-ingestion-mapping-command.md)                                                                    | Administrar la asignación de ingesta        |
| [`.create tables`](create-tables-command.md), [`.create table`](create-table-command.md), [`.alter table`](alter-table-command.md), [`.alter-merge table`](alter-table-command.md), [`.drop tables`](drop-table-command.md), [`.drop table`](drop-table-command.md), [`.undo drop table`](undo-drop-table-command.md), [`.rename table`](rename-table-command.md) | Crear/modificar/quitar tablas       |
| [`.show tables`](show-tables-command.md) [`.show table details`](show-table-details-command.md)[`.show table schema`](show-table-schema-command.md)                                                                                      | Enumerar tablas en una base de datos  |
| `.ingest`, `.set` , `.append` , `.set-or-append` (consulte [ingesta de datos](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands) para obtener más información)).                                                                                                                                                                                      | Ingesta de datos en una tabla     |

## <a name="crud-naming-conventions-for-tables"></a>Convenciones de nomenclatura CRUD para tablas 
(Consulte los detalles completos en las secciones vinculadas a en la tabla anterior).
 
| Sintaxis de comandos                             | Semántica                                                                                                             |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| `.create entityType entityName ...`        | Si existe una entidad de ese tipo y nombre, devuelve la entidad. De lo contrario, cree la entidad.                          |
| `.create-merge entityType entityName...`   | Si existe una entidad de ese tipo y nombre, combine la entidad existente con la entidad especificada. De lo contrario, cree la entidad. |
| `.alter entityType entityName ...`         | Si no existe una entidad de ese tipo y nombre, error. De lo contrario, reemplácelo por la entidad especificada.            |
| `.alter-merge entityType entityName ...`   | Si no existe una entidad de ese tipo y nombre, error. De lo contrario, mezcle con la entidad especificada.              |
| `.drop entityType entityName ...`          | Si no existe una entidad de ese tipo y nombre, error. En caso contrario, colóquelo.                                         |
| `.drop entityType entityName ifexists ...` | Si una entidad de ese tipo y nombre no existe, se devuelve. En caso contrario, colóquelo.                                        |
 
> [!NOTE]
> "Merge" es una combinación lógica de dos entidades:
>
> * Si se define una propiedad para una entidad, pero no para la otra, aparece con su valor original en la entidad combinada.
> * Si se define una propiedad para ambas entidades y tiene el mismo valor en ambos, aparece una vez con ese valor en la entidad combinada.
> * Si se define una propiedad para ambas entidades pero tiene valores diferentes, se produce un error.
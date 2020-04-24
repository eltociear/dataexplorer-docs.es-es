---
title: 'Administración de tablas: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la administración de tablas en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/27/2020
ms.openlocfilehash: 7b7e4c5c7111354864aa939ece76be2ab0a8ac15
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519574"
---
# <a name="tables-management"></a>Gestión de tablas

En este tema se describe el ciclo de vida de las tablas y los comandos de control asociados.

Seleccione los enlaces de la tabla siguiente para obtener más información sobre ellos.

| Comandos:                                                                                                                 | Operación                       |
|--------------------------------------------------------------------------------------------------------------------------|---------------------------------|
| [`.alter table docstring`](alter-table-docstring-command.md), [`.alter table folder`](alter-table-folder-command.md)                                                                                                                                                                                                   | Administrar las propiedades de visualización de tablas |
| [`.create ingestion mapping`](create-ingestion-mapping-command.md), [`.show ingestion mappings`](show-ingestion-mapping-command.md), [`.alter ingestion mapping`](alter-ingestion-mapping-command.md), [`.drop ingestion mapping`](drop-ingestion-mapping-command.md)                                                                    | Administrar la asignación de ingesta        |
| [`.create tables`](create-tables-command.md), [`.create table`](create-table-command.md), [`.alter table`](alter-table-command.md), [`.alter-merge table`](alter-table-command.md), [`.drop tables`](drop-table-command.md), [`.drop table`](drop-table-command.md), [`.undo drop table`](undo-drop-table-command.md), [`.rename table`](rename-table-command.md) | Crear/modificar/soltar tablas       |
| [`.show tables`](show-tables-command.md) [`.show table details`](show-table-details-command.md)[`.show table schema`](show-table-schema-command.md)                                                                                      | Enumerar tablas en una base de datos  |
| `.ingest`, `.set` `.append`, `.set-or-append` , (consulte [Ingestión de](./data-ingestion/index.md) datos para obtener más información).)                                                                                                                                                                                      | Ingestión de datos en una tabla     |

## <a name="crud-naming-conventions-for-tables"></a>Convenciones de nomenclatura CRUD para tablas 
(Consulte todos los detalles en las secciones vinculadas a la tabla anterior.)
 
| Sintaxis de comandos                             | Semántica                                                                                                             |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| `.create entityType entityName ...`        | Si existe una entidad de ese tipo y nombre, devuelve la entidad. De lo contrario, cree la entidad.                          |
| `.create-merge entityType entityName...`   | Si existe una entidad de ese tipo y nombre, combine la entidad existente con la entidad especificada. De lo contrario, cree la entidad. |
| `.alter entityType entityName ...`         | Si no existe una entidad de ese tipo y nombre, error. De lo contrario, sustitúyalo por la entidad especificada.            |
| `.alter-merge entityType entityName ...`   | Si no existe una entidad de ese tipo y nombre, error. De lo contrario, combínelo con la entidad especificada.              |
| `.drop entityType entityName ...`          | Si no existe una entidad de ese tipo y nombre, error. De lo contrario, suéltalo.                                         |
| `.drop entityType entityName ifexists ...` | Si no existe una entidad de ese tipo y nombre, devuelva. De lo contrario, suéltalo.                                        |
 
> [!NOTE]
> "Combinar" es una combinación lógica de dos entidades:
>
> * Si se define una propiedad para una entidad pero no para la otra, aparece con su valor original en la entidad combinada.
> * Si se define una propiedad para ambas entidades y tiene el mismo valor en ambas, aparece una vez con ese valor en la entidad combinada.
> * Si se define una propiedad para ambas entidades pero tiene valores diferentes, se genera un error.
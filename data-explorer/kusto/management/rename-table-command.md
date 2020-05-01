---
title: '. cambie el nombre de la tabla y. cambie el nombre de las tablas: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. cambiar el nombre de tablas y cambiar el nombre de las tablas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: a1644021d1e2df294782d3b24e111d3c57af5f91
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617585"
---
# <a name="rename-table-and-rename-tables"></a>. Rename Table y. Rename tables

Cambia el nombre de una tabla existente.

El `.rename tables` comando cambia el nombre de un número de tablas en la base de datos como una sola transacción.

Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.rename``table` *OldName* OldName `to` ( *NewName* )

`.rename``tables` *NewName*NewName = *OldName* [`ifexists`] [`,` ...]

> [!NOTE]
> * *OldName* es el nombre de una tabla existente. Se produce un error y todo el comando produce un error (no tiene ningún efecto) si *OldName* no denomina una tabla existente `ifexists` , a menos que se especifique (en cuyo caso se omite esta parte del comando Rename).
> * *NewName* es el nuevo nombre de la tabla existente que solía denominarse *OldName*.
> * Si `ifexists` se especifica, modifica el comportamiento del comando para omitir el cambio de nombre de partes de tablas que no existen.

**Comentarios:**

Este comando solo funciona en las tablas de la base de datos en el ámbito.
Los nombres de tabla no se pueden calificar con nombres de clúster o de base de datos.

Este comando no crea nuevas tablas ni quita las tablas existentes.
La transformación descrita por el comando debe ser tal que el número de tablas de la base de datos no cambie.

El comando **admite** el intercambio de nombres de tabla o permutaciones más complejas, siempre y cuando cumplan las reglas anteriores. Por ejemplo, ingerir datos en varias tablas de almacenamiento provisional y, a continuación, intercambiarlas con tablas existentes en una sola transacción.

**Ejemplos**

Imagine una base de datos con las tablas `A`siguientes `B`: `C`,, `A_TEMP`y.
`A` El siguiente comando intercambiará `A_TEMP` y (de modo `A_TEMP` que se llamará `A`a la tabla ahora y viceversa), cambiará el `B` nombre `NEWB`a y mantendrá `C` tal cual. 

```kusto
.rename tables A=A_TEMP, NEWB=B, A_TEMP=A
``` 

La siguiente secuencia de comandos:
1. Crea una nueva tabla temporal
1. Reemplaza una tabla existente o no existente por la nueva tabla

```kusto
// Drop the temporary table if it exists
.drop table TempTable ifexists

// Create a new table
.set TempTable <| ...

// Swap the two tables
.rename tables TempTable=Table ifexists, Table=TempTable

// Drop the temporary table (which used to be Table) if it exists
.drop table TempTable ifexists
```

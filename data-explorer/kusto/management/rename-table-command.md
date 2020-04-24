---
title: Tabla .rename y tablas .rename- Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las tablas .rename y las tablas .rename en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: db3e38c562573f52df70979b71fe72d8947158bf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520492"
---
# <a name="rename-table-and-rename-tables"></a>Tabla .rename y tablas .rename

Cambia el nombre de una tabla existente.

El `.rename tables` comando cambia el nombre de varias tablas de la base de datos como una sola transacción.

Requiere [permiso](../management/access-control/role-based-authorization.md)de administrador de base de datos .

**Sintaxis**

`.rename``table` *OldName* `to` *NewName*

`.rename``tables` *NewName* = *OldName* [`ifexists`] [`,` ...]

> [!NOTE]
> * *OldName* es el nombre de una tabla existente. Se genera un error y se produce un error en todo el comando `ifexists` (no tiene ningún efecto) si *OldName* no nombra una tabla existente, a menos que se especifique (en cuyo caso se omite esta parte del comando rename).
> * *NewName* es el nuevo nombre de la tabla existente que solía llamarse *OldName*.
> * Si `ifexists` se especifica, modifica el comportamiento del comando para omitir el cambio de nombre de las partes de tablas inexistentes.

**Comentarios:**

Este comando solo funciona en tablas de la base de datos en el ámbito.
Los nombres de tabla no se pueden calificar con nombres de clúster o base de datos.

Este comando no crea nuevas tablas ni elimina las tablas existentes.
La transformación descrita por el comando debe ser tal que el número de tablas de la base de datos no cambie.

El **comando** admite el intercambio de nombres de tabla, o permutaciones más complejas, siempre y cuando se adhieran a las reglas anteriores. Por ejemplo, ingiese datos en varias tablas de ensayo y, a continuación, intercambielos con tablas existentes en una sola transacción.

**Ejemplos**

Imagine una base de `A`datos `B` `C`con `A_TEMP`las siguientes tablas: , , , y .
El siguiente comando `A` `A_TEMP` intercambiará y `A_TEMP` (para que `A`ahora se llame a `B` `NEWB`la tabla `C` , y al revés), cambie el nombre a , y mantenga tal cual. 

```
.rename tables A=A_TEMP, NEWB=B, A_TEMP=A
``` 

La siguiente secuencia de comandos:
1. Crea una nueva tabla temporal
1. Reemplaza una tabla existente o no existente por la nueva tabla

```
// Drop the temporary table if it exists
.drop table TempTable ifexists

// Create a new table
.set TempTable <| ...

// Swap the two tables
.rename tables TempTable=Table ifexists, Table=TempTable

// Drop the temporary table (which used to be Table) if it exists
.drop table TempTable ifexists
```
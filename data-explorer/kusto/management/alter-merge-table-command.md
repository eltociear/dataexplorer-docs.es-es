---
title: '. Alter-Merge Table: Azure Explorador de datos'
description: En este artículo se describe el comando de tabla. Alter-Merge.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/08/2020
ms.openlocfilehash: cc4002d9af8b18841714ac9f91809fb18274782f
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84670521"
---
# <a name="alter-merge-table"></a>. Alter-Merge Table
 
El comando `.alter-merge table`:

* Protege los datos de las columnas existentes
* Agrega nuevas columnas, `docstring` , y a una tabla existente.
* Se debe ejecutar en el contexto de una base de datos específica que alcance el nombre de la tabla.
* Requiere [permiso de administrador de tabla](../management/access-control/role-based-authorization.md)

> [!WARNING]
> El uso `.alter-merge` incorrecto del comando puede provocar la pérdida de datos.

> [!TIP]
> `.alter-merge`Tiene un homólogo, el `.alter` comando de tabla que tiene una funcionalidad similar. Para obtener más información, vea [. ALTER TABLE](../management/alter-table-command.md)

**Sintaxis**

`.alter-merge``table` *TableName* (*columnName*:*columnType*,...)  [ `with` `(` [ `docstring` `=` *Documentación*] [ `,` `folder` `=` *nombrecarpeta*] `)` ]

Especifique las columnas de la tabla:
 * Las columnas que no existen y que se especifican se agregan al final del esquema existente.
 * Si el esquema que se pasa no contiene algunas columnas de la tabla, las columnas no se eliminarán.
 * Si especifica una columna existente con un tipo diferente, se producirá un error en el comando.

> [!TIP]
> Utilice `.show table [TableName] cslschema` para obtener el esquema de columna existente antes de modificarlo.

¿Cómo afectará el comando a los datos?
* El comando no modifica físicamente los datos existentes. Los datos de las columnas eliminadas se omiten. Se supone que los datos de las nuevas columnas son NULL.
* En función de cómo esté configurado el clúster, la ingesta de datos podría modificar el esquema de columnas de la tabla, incluso sin la interacción del usuario. Cuando realice cambios en el esquema de columna de una tabla, asegúrese de que la ingesta no agregará las columnas necesarias que el comando quitará.

**Ejemplos**

```kusto
.alter-merge table MyTable (ColumnX:string, ColumnY:int) 
.alter-merge table MyTable (ColumnX:string, ColumnY:int) with (docstring = "Some documentation", folder = "Folder1")
```
 

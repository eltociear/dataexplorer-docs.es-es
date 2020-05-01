---
title: '. ALTER TABLE y. Alter-Merge Table: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. ALTER TABLE y la tabla. Alter-Merge en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/17/2020
ms.openlocfilehash: 18b0502754e95ca56d6a7f6946b9330bd42b174a
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616446"
---
# <a name="alter-table-and-alter-merge-table"></a>. ALTER TABLE y. Alter-Merge Table

El `.alter table` comando establece un nuevo esquema de columna, DocString y una carpeta en una tabla existente, invalidando el esquema de columna existente, el DocString y la carpeta. Se conservan los datos de las columnas existentes que se "conservan" mediante el comando (por ejemplo, se puede usar este comando para reordenar las columnas de una tabla).

El `.alter-merge table` comando agrega nuevas columnas, DocString y Folder, a una tabla existente.
Se conservan los datos de las columnas existentes.

Ambos comandos se deben ejecutar en el contexto de una base de datos específica que alcance el nombre de la tabla.

Requiere [permiso de administrador de tablas](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.alter``table` *TableName* (*columnName*:*columnType*,...)  [`with` `(`[`docstring` `=` *FolderName* *Documentation*`,` Documentación] [ `folder` NombreCarpeta] `)`] `=`

`.alter-merge``table` *TableName* (*columnName*:*columnType*,...)  [`with` `(`[`docstring` `=` *FolderName* *Documentation*`,` Documentación] [ `folder` NombreCarpeta] `)`] `=`

Especifique las columnas que la tabla debe tener después de completarse correctamente. 

> [!WARNING]
> El uso `.alter` incorrecto del comando puede provocar la pérdida de datos.
> Lea atentamente las diferencias `.alter` entre `.alter-merge` y por debajo.

`.alter-merge`:

 * Las columnas que no existen y que se especifican se agregan al final del esquema existente.
 * Si el esquema que se pasa no contiene algunas columnas de la tabla, no se eliminarán.
 * Si especificó una columna existente con un tipo diferente, se producirá un error en el comando.

`.alter`solo (no `.alter-merge`):

 * La tabla tendrá exactamente las mismas columnas, en el mismo orden, según lo especificado.
 * Las columnas existentes que no se especifican en el comando se quitarán ( `.drop column`como en) y se perderán los datos que contengan.
 * No se admite la modificación de un tipo de columna al modificar una tabla. En su lugar, use el comando [. Alter Column](alter-column.md) .

> [!TIP] 
> Utilice `.show table [TableName] cslschema` para obtener el esquema de columna existente antes de modificarlo. 

Lo siguiente se aplica a ambos comandos:

1. Estos comandos no modifican físicamente los datos existentes. Los datos de las columnas eliminadas se omiten. Se supone que los datos de las nuevas columnas son NULL.
1. En función de cómo esté configurado el clúster, la ingesta de datos podría modificar el esquema de columnas de la tabla, incluso sin la interacción del usuario. Por lo tanto, al realizar cambios en el esquema de columna de una tabla, asegúrese de que la ingesta no agregará las columnas necesarias que el comando quitará.

> [!WARNING]
> Los procesos de ingesta de datos en la tabla que modifican el esquema de columnas de la tabla, `.alter table` que se producen en paralelo con el comando, se pueden realizar de forma independiente en el orden de las columnas de la tabla. También hay un riesgo de que los datos se ingestan en columnas incorrectas. Para evitarlo, detenga la ingesta durante estos comandos o asegúrese de que dichas operaciones de ingesta siempre utilizan un objeto de asignación.

**Ejemplos**

```kusto
.alter table MyTable (ColumnX:string, ColumnY:int) 
.alter table MyTable (ColumnX:string, ColumnY:int) with (docstring = "Some documentation", folder = "Folder1")
.alter-merge table MyTable (ColumnX:string, ColumnY:int) 
.alter-merge table MyTable (ColumnX:string, ColumnY:int) with (docstring = "Some documentation", folder = "Folder1")
```
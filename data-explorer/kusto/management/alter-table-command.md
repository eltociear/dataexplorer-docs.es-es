---
title: 'Tabla .alter y tabla .alter-merge: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la tabla .alter y la tabla .alter-merge en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/17/2020
ms.openlocfilehash: 516c5c7b85f7c310188fd11ae24e52cb23423143
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522379"
---
# <a name="alter-table-and-alter-merge-table"></a>Tabla .alter y tabla .alter-merge

El `.alter table` comando establece un nuevo esquema de columna, docstring y carpeta en una tabla existente, reemplazando el esquema de columna existente, docstring y carpeta. Los datos de las columnas existentes que son "conservados" por el comando se conservan (por lo que este comando se puede utilizar, por ejemplo, para reordenar las columnas de una tabla).

El `.alter-merge table` comando agrega nuevas columnas, docstring y folder, a una tabla existente.
Se conservan los datos de las columnas existentes.

Ambos comandos deben ejecutarse en el contexto de una base de datos específica que ámbito el nombre de la tabla.

Requiere [permiso de administrador de tabla](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.alter``table` *TableName* (*columnName*:*columnType*, ...)  [`with` `(``docstring` `=` [ *Documentación*`,` `folder` `=` ] `)`[ *NombreCarpeta*] ]

`.alter-merge``table` *TableName* (*columnName*:*columnType*, ...)  [`with` `(``docstring` `=` [ *Documentación*`,` `folder` `=` ] `)`[ *NombreCarpeta*] ]

Especifique las columnas que la tabla debe tener después de completarse correctamente. 

> [!WARNING]
> El `.alter` uso incorrecto del comando puede provocar la pérdida de datos.
> Lea atentamente `.alter` las `.alter-merge` diferencias entre y por debajo.

`.alter-merge`:

 * Las columnas que no existen y que especifique se agregan al final del esquema existente.
 * Si el esquema pasado no contiene algunas columnas de tabla, no se eliminarán.
 * Si especificó una columna existente con un tipo diferente, se producirá un error en el comando.

`.alter`solamente `.alter-merge`(no):

 * La tabla tendrá exactamente las mismas columnas, en el mismo orden, como se especifica.
 * Las columnas existentes que no se especifican en `.drop column`el comando se quitarán (como en ) y se perderán los datos en ellas.
 * No se admite la modificación de un tipo de columna al modificar una tabla. Utilice el comando [de columna .alter](alter-column.md) en su lugar.

> [!TIP] 
> Se `.show table [TableName] cslschema` usa para obtener el esquema de columna existente antes de modificarlo. 

Lo siguiente se aplica a ambos comandos:

1. Estos comandos no modifican físicamente los datos existentes. Los datos de las columnas eliminadas se omiten. Se supone que los datos de las columnas nuevas son null.
1. Dependiendo de cómo se configure el clúster, la ingesta de datos podría modificar el esquema de columna de la tabla, incluso sin la interacción del usuario. Por lo tanto, al realizar cambios en el esquema de columna de una tabla, asegúrese de que la ingesta no agregará las columnas necesarias que el comando quitará.

> [!WARNING]
> Los procesos de ingesta de datos en la tabla que modifican el esquema de columna de la tabla, que se producen en paralelo con el `.alter table` comando, se pueden realizar agnósticos al orden de las columnas de la tabla. También existe el riesgo de que los datos se ingienden en las columnas incorrectas. Para evitarlo, detenga la ingesta durante estos comandos o asegúrese de que dichas operaciones de ingesta siempre utilizan un objeto de asignación.

**Ejemplos**

```
.alter table MyTable (ColumnX:string, ColumnY:int) 
.alter table MyTable (ColumnX:string, ColumnY:int) with (docstring = "Some documentation", folder = "Folder1")
.alter-merge table MyTable (ColumnX:string, ColumnY:int) 
.alter-merge table MyTable (ColumnX:string, ColumnY:int) with (docstring = "Some documentation", folder = "Folder1")
```
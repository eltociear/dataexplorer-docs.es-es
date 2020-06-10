---
title: '. ALTER TABLE: Azure Explorador de datos'
description: En este artículo se describe el comando. ALTER TABLE.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: reference
ms.date: 06/08/2020
ms.openlocfilehash: 29f0c65a635b6e4fe6ffe3288cc1dcdde702fc8a
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84665034"
---
# <a name="alter-table"></a>.alter table
 
El comando `.alter table`:
* Protege los datos en columnas "conservadas"
* Reordena las columnas de la tabla
* Establece un nuevo esquema de columna, `docstring` , y en una tabla existente, sobrescribiendo el esquema de columna existente, `docstring` y la carpeta
* Se debe ejecutar en el contexto de una base de datos específica que alcance el nombre de la tabla.
* Requiere [permiso de administrador de tabla](../management/access-control/role-based-authorization.md)

> [!WARNING]
> El uso `.alter` incorrecto del comando puede provocar la pérdida de datos.

> [!TIP]
> `.alter`Tiene un homólogo, el `.alter-merge` comando de tabla que tiene una funcionalidad similar. Para obtener más información, vea [. Alter-Merge Table](../management/alter-merge-table-command.md)

**Sintaxis**

`.alter``table` *TableName* (*columnName*:*columnType*,...)  [ `with` `(` [ `docstring` `=` *Documentación*] [ `,` `folder` `=` *nombrecarpeta*] `)` ]


 * La tabla tendrá exactamente las mismas columnas, en el mismo orden, según lo especificado.
 Especifique las columnas de la tabla:
 * Si no se especifican columnas existentes en el comando, se quitarán y se perderán los datos que contengan, como sucede con el `.drop column` comando.
 * Cuando se modifica una tabla, no se admite la modificación de un tipo de columna. En su lugar, use el comando [. Alter Column](alter-column.md) .

> [!TIP]
> Utilice `.show table [TableName] cslschema` para obtener el esquema de columna existente antes de modificarlo.


¿Cómo afectará el comando a los datos?
* El comando no modifica físicamente los datos existentes. Los datos de las columnas eliminadas se omiten. Se supone que los datos de las nuevas columnas son NULL.
* En función de cómo esté configurado el clúster, la ingesta de datos podría modificar el esquema de columnas de la tabla, incluso sin la interacción del usuario. Cuando realice cambios en el esquema de columna de una tabla, asegúrese de que la ingesta no agregará las columnas necesarias que el comando quitará.

> [!WARNING]
> Los procesos de ingesta de datos en la tabla que modifican el esquema de columnas de la tabla y que se producen en paralelo con el `.alter table` comando, se pueden realizar de forma independiente en el orden de las columnas de la tabla. También hay un riesgo de que los datos se ingestan en columnas incorrectas. Para evitar estos problemas, detenga la ingesta durante el comando o asegúrese de que dichas operaciones de ingesta siempre utilizan un objeto de asignación.

**Ejemplos**

```kusto
.alter table MyTable (ColumnX:string, ColumnY:int) 
.alter table MyTable (ColumnX:string, ColumnY:int) with (docstring = "Some documentation", folder = "Folder1")
```

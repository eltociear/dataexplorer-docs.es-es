---
title: . ALTER TABLE DocString-Azure Explorador de datos
description: En este artículo se describe el `.alter table docstring` comando de Azure explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 65b71ab15763506683c461f04975d22d396f6405
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512544"
---
# <a name="alter-table-docstring"></a>.alter table docstring

Modifica el valor de DocString de una tabla existente.

`.alter``table` *TableName* `docstring` *Documentación* de TableName

> [!NOTE]
> * Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md)
> * Se permite que el [usuario de base de datos](../management/access-control/role-based-authorization.md) que creó originalmente la tabla lo modifique
> * Si la tabla no existe, se devuelve un error. Para crear una tabla nueva, vea [. CREATE TABLE](create-table-command.md)

**Ejemplo** 

```kusto
.alter table LyricsAsTable docstring "This is the theme to Garry's show"
```
 

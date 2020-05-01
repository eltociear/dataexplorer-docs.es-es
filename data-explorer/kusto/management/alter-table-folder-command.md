---
title: . ALTER TABLE Folder-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe la carpeta. ALTER TABLE de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/06/2020
ms.openlocfilehash: f03a2927d3f0a4fe7ee1719594f2d1f19e231d0f
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618367"
---
# <a name="alter-table-folder"></a>.alter table folder

Modifica el valor de carpeta de una tabla existente. 

`.alter``table` *TableName* `folder` *Carpeta* TableName

> [!NOTE]
> * Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md)
> * También se permite que el [usuario de base de datos](../management/access-control/role-based-authorization.md) que creó originalmente la tabla lo edite
> * Si la tabla no existe, se devuelve un error. Para crear una nueva tabla, vea [. CREATE TABLE](create-table-command.md)

**Ejemplos** 

```kusto
.alter table MyTable folder "Updated folder"
```

```kusto
.alter table MyTable folder @"First Level\Second Level"
```
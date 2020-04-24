---
title: 'Carpeta de la tabla .alter: Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la carpeta de tabla .alter en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/06/2020
ms.openlocfilehash: a1d368d50f0e42acdbc25348b8025fbe8b530536
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522311"
---
# <a name="alter-table-folder"></a>.alter table folder

Altera el valor De carpeta de una tabla existente. 

`.alter``table` *Carpeta* *TableName* `folder`

> [!NOTE]
> * Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos
> * El usuario de [la base](../management/access-control/role-based-authorization.md) de datos que creó originalmente la tabla también puede editarla
> * Si la tabla no existe, se devuelve un error. Para crear una nueva tabla, consulte [Tabla .create](create-table-command.md)

**Ejemplos** 

```
.alter table MyTable folder "Updated folder"
```

```
.alter table MyTable folder @"First Level\Second Level"
```
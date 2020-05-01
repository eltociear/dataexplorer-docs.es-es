---
title: . ALTER TABLE DocString-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe. ALTER TABLE DocString en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 790cd9805be6dd4440ef2eb51c504044dc069b3c
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617789"
---
# <a name="alter-table-docstring"></a>.alter table docstring

Modifica el valor de DocString de una tabla existente.

`.alter``table` *TableName* `docstring` *Documentación* de TableName

> [!NOTE]
> * Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md)
> * También se permite la modificación de la tabla al [usuario](../management/access-control/role-based-authorization.md) de la base de datos que creó originalmente la tabla
> * Si la tabla no existe, se devuelve un error. Para crear una tabla nueva, vea [. CREATE TABLE](create-table-command.md)

**Ejemplo** 

```kusto
.alter table LyricsAsTable docstring "This is the theme to Garry's show"
```
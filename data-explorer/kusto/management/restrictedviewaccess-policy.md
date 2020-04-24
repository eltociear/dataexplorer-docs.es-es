---
title: 'Directiva RestrictedViewAccess: Explorador de azure Data Explorer ( Microsoft Docs'
description: En este artículo se describe la directiva RestrictedViewAccess en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 9fcf37d30bfe3ab0f9c4b5d4a720e6a5ba4ffe34
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520458"
---
# <a name="restrictedviewaccess-policy"></a>Directiva RestrictedViewAccess

La directiva *RestrictedViewAccess* se documenta [aquí.](../management/restrictedviewaccesspolicy.md)

Los comandos de control para habilitar o deshabilitar la directiva en las tablas de la base de datos son los siguientes:

Para activar/desactivar la directiva:
```kusto
.alter table TableName policy restricted_view_access true|false
```

Para habilitar/deshabilitar la directiva de varias tablas:
```kusto
.alter tables (TableName1, ..., TableNameN) policy restricted_view_access true|false
```

Para ver la directiva:
```kusto
.show table TableName policy restricted_view_access  

.show table * policy restricted_view_access  
```

Para eliminar la directiva (equivalente a deshabilitarla):
```kusto
.delete table TableName policy restricted_view_access  
```
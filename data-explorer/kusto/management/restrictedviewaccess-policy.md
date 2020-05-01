---
title: 'Administración de directivas de Kusto RestrictedViewAccess: Azure Explorador de datos'
description: En este artículo se describe la Directiva RestrictedViewAccess en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 9da59a53819396cf2cbd522f4a1e1296f585bf2f
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617554"
---
# <a name="restrictedviewaccess-policy"></a>Directiva RestrictedViewAccess

La directiva *RestrictedViewAccess* se documenta [aquí](../management/restrictedviewaccesspolicy.md).

Los comandos de control para habilitar o deshabilitar la Directiva en las tablas de la base de datos son los siguientes:

Para habilitar o deshabilitar la Directiva:
```kusto
.alter table TableName policy restricted_view_access true|false
```

Para habilitar o deshabilitar la Directiva de varias tablas:
```kusto
.alter tables (TableName1, ..., TableNameN) policy restricted_view_access true|false
```

Para ver la Directiva:
```kusto
.show table TableName policy restricted_view_access  

.show table * policy restricted_view_access  
```

Para eliminar la Directiva (equivalente a deshabilitar):
```kusto
.delete table TableName policy restricted_view_access  
```
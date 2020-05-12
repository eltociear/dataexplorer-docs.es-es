---
title: 'current_principal (): Explorador de datos de Azure'
description: En este artículo se describe current_principal () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: b1d45fb8b0749a4be30854dd9b0120a7eb127bf2
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227305"
---
# <a name="current_principal"></a>current_principal()

::: zone pivot="azuredataexplorer"

Devuelve el nombre de la entidad de seguridad actual que ejecuta la consulta.

**Sintaxis**

`current_principal()`

**Devuelve**

Nombre completo de la entidad de seguridad actual (FQN) como un `string` .  
La cadena tiene el siguiente formato:  
*PrinciplaType* `=` *PrincipalId* `;` *TenantId*

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print fqn=current_principal()
```

|FQN|
|---|
|aaduser = 346e950e-4a62-42bf-96f5-4cf4eac3f11e; 72f988bf-86f1-41AF-91ab-2d7cd011db47|

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

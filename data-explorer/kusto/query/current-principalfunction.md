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
ms.openlocfilehash: 43e92edd74861acc8207a855243f9ec1e012070a
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717366"
---
# <a name="current_principal"></a>current_principal()

::: zone pivot="azuredataexplorer"

Devuelve el nombre de la entidad de seguridad actual que ejecuta la consulta.

**Sintaxis**

`current_principal()`

**Devuelve**

Nombre completo de la entidad de seguridad actual (FQN) como `string` .  
El formato de cadena es:  
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

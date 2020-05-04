---
title: current_principal ()-Explorador de datos de Azure | Microsoft Docs
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
ms.openlocfilehash: 0561ac200105015e6d1c1cce1c16fe5f60fc2ccf
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737715"
---
# <a name="current_principal"></a>current_principal()

::: zone pivot="azuredataexplorer"

Devuelve el nombre de la entidad de seguridad actual que ejecuta la consulta.

**Sintaxis**

`current_principal()`

**Devuelve**

Nombre completo de la entidad de seguridad actual (FQN) como un `string`.  
La cadena tiene el siguiente formato:  
*PrinciplaType*`=`*PrincipalId*PrincipalId`;`*TenantId*

**Ejemplo**

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

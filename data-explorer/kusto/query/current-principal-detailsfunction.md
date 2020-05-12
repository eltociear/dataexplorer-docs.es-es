---
title: 'current_principal_details (): Explorador de datos de Azure'
description: En este artículo se describe current_principal_details () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/08/2019
ms.openlocfilehash: f71770d2cc9d44987731a247fa8eb945ed323391
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227524"
---
# <a name="current_principal_details"></a>current_principal_details()

Devuelve los detalles de la entidad de seguridad que ejecuta la consulta.

**Sintaxis**

`current_principal_details()`

**Devuelve**

Detalles de la entidad de seguridad actual como un `dynamic` .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print d=current_principal_details()
```

|d|
|---|
|{<br>  "UserPrincipalName": " user@fabrikam.com ",<br>  "IdentityProvider": " https://sts.windows.net ",<br>  "Autoridad": "72f988bf-86f1-41AF-91ab-2d7cd011db47",<br>  "MFA": "true",<br>  "Type": "AadUser",<br>  "DisplayName": "James Smith (UPN: user@fabrikam.com )",<br>  "ObjectId": "346e950e-4a62-42bf-96f5-4cf4eac3f11e",<br>  "FQN": null,<br>  "Notas": null<br>}|

---
title: 'current_principal_is_member_of (): Explorador de datos de Azure'
description: En este artículo se describe current_principal_is_member_of () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/09/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 521165f5b0af31207d587f3d9514e7538d284258
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227348"
---
# <a name="current_principal_is_member_of"></a>current_principal_is_member_of()

::: zone pivot="azuredataexplorer"

Comprueba la pertenencia al grupo o la identidad principal de la entidad de seguridad actual que ejecuta la consulta.

```kusto
print current_principal_is_member_of(
    'aaduser=user1@fabrikam.com', 
    'aadgroup=group1@fabrikam.com',
    'aadapp=66ad1332-3a94-4a69-9fa2-17732f093664;72f988bf-86f1-41af-91ab-2d7cd011db47'
    )
```

**Sintaxis**

`current_principal_is_member_of`(`*list of string literals*`)

**Argumentos**

* *lista de expresiones* : una lista separada por comas de literales de cadena, donde cada literal es una cadena de nombre completo (FQN) de entidad de seguridad formada como:  
*PrinciplaType* `=` *PrincipalId* `;` *TenantId*

| PrincipalType   | Prefijo FQN  |
|-----------------|-------------|
| Usuario de AAD        | `aaduser=`  |
| Grupo de AAD       | `aadgroup=` |
| Aplicación AAD | `aadapp=`   |

**Devuelve**

La función devuelve:
* `true`: Si la entidad de seguridad actual que ejecuta la consulta coincidía correctamente con al menos un argumento de entrada.
* `false`: Si la entidad de seguridad actual que ejecuta la consulta no era miembro de ningún `aadgroup=` argumento FQN y no es igual a ninguno de los `aaduser=` `aadapp=` argumentos o FQN.
* `(null)`: Si la entidad de seguridad actual que ejecuta la consulta no era miembro de ningún `aadgroup=` argumento de FQN y no es igual a ninguno de los `aaduser=` `aadapp=` argumentos o FQN, y al menos un argumento de FQN no se resolvió correctamente (no se adpresa en AAD). 

> [!NOTE]
> Dado que la función devuelve un valor de tres Estados ( `true` , `false` y `null` ), es importante depender solo de los valores devueltos positivos para confirmar la pertenencia correcta. En otras palabras, las siguientes expresiones no son las mismas:
> 
> * `where current_principal_is_member_of('non-existing-group')`
> * `where current_principal_is_member_of('non-existing-group') != false` 


**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print result=current_principal_is_member_of(
    'aaduser=user1@fabrikam.com', 
    'aadgroup=group1@fabrikam.com',
    'aadapp=66ad1332-3a94-4a69-9fa2-17732f093664;72f988bf-86f1-41af-91ab-2d7cd011db47'
    )
```

| resultado |
|--------|
| (null) |

Usar la matriz dinámica en lugar de varios argumentos:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print result=current_principal_is_member_of(
    dynamic([
    'aaduser=user1@fabrikam.com', 
    'aadgroup=group1@fabrikam.com',
    'aadapp=66ad1332-3a94-4a69-9fa2-17732f093664;72f988bf-86f1-41af-91ab-2d7cd011db47'
    ]))
```

| resultado |
|--------|
| (null) |

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

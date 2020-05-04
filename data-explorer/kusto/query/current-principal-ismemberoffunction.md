---
title: current_principal_is_member_of ()-Explorador de datos de Azure | Microsoft Docs
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
ms.openlocfilehash: 4b6f7d0b9ab4074f16ca00b4a3febb1a17351736
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737766"
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
*PrinciplaType*`=`*PrincipalId*PrincipalId`;`*TenantId*

| PrincipalType   | Prefijo FQN  |
|-----------------|-------------|
| Usuario de AAD        | `aaduser=`  |
| Grupo de AAD       | `aadgroup=` |
| Aplicación AAD | `aadapp=`   |

**Devuelve**

La función devuelve:
* `true`: Si la entidad de seguridad actual que ejecuta la consulta coincidía correctamente con al menos un argumento de entrada.
* `false`: Si la entidad de seguridad actual que ejecuta la consulta no era `aadgroup=` miembro de ningún argumento FQN y no es igual a `aaduser=` ninguno `aadapp=` de los argumentos o FQN.
* `(null)`: Si la entidad de seguridad actual que ejecuta la consulta no era `aadgroup=` miembro de ningún argumento de FQN y no es igual `aaduser=` a `aadapp=` ninguno de los argumentos o FQN, y al menos un argumento de FQN no se resolvió correctamente (no se adpresa en AAD). 

> [!NOTE]
> Dado que la función devuelve un valor de tres Estados`true`( `false`, y `null`), es importante depender solo de los valores devueltos positivos para confirmar la pertenencia correcta. En otras palabras, las siguientes expresiones no son las mismas:
> 
> * `where current_principal_is_member_of('non-existing-group')`
> * `where current_principal_is_member_of('non-existing-group') != false` 


**Ejemplo**

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

Usar matrices dinámicas en lugar de argumentos múltiple:

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

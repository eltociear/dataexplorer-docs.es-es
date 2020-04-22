---
title: current_principal_is_member_of() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe current_principal_is_member_of() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/09/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 03d565c9eb61703b326a01b31bb6b1a79742f006
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766055"
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

* *lista de expresiones* - una lista separada por comas de literales de cadena, donde cada literal es una cadena principal de nombre completo (FQN) formada como:  
*PrinciplaType*`=`*PrincipalId*`;`*TenantId*

| PrincipalType   | Prefijo FQN  |
|-----------------|-------------|
| Usuario de AAD        | `aaduser=`  |
| Grupo AAD       | `aadgroup=` |
| Aplicación aAAD | `aadapp=`   |

**Devuelve**

La función devuelve:
* `true`: si la entidad de seguridad actual que ejecuta la consulta se ha emparejado correctamente para al menos un argumento de entrada.
* `false`: si la entidad de seguridad actual `aadgroup=` que ejecuta la consulta no era `aaduser=` miembro `aadapp=` de ningún argumento FQN y no era igual a ninguno de los argumentos o FQN.
* `(null)`: si la entidad de seguridad actual `aadgroup=` que ejecuta la consulta no era `aaduser=` miembro `aadapp=` de ningún argumento FQN y no era igual a ninguno de los argumentos o FQN, y al menos un argumento FQN no se resolvió correctamente (no se presionó en AAD). 

> [!NOTE]
> Dado que la función devuelve`true` `false`un `null`valor de estado tri- ( , , y ), es importante confiar solo en valores devueltos positivos para confirmar la pertenencia correcta. En otras palabras, las siguientes expresiones NO son las mismas:
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

Uso de matrices dinámicas en lugar de argumentos multple:

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

Esto no se admite en Azure Monitor

::: zone-end

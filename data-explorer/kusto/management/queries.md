---
title: 'Administración de consultas: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la administración de consultas en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
ms.openlocfilehash: b888aa004b4c8b02cd4e1d130bb0b5319af018b3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520526"
---
# <a name="queries-management"></a>Gestión de consultas

## <a name="show-queries"></a>Consultas .show

El `.show` `queries` comando devuelve una lista de consultas que han alcanzado un estado final y que el usuario que invoca el comando tiene acceso para ver:


* Un administrador de base de datos o un monitor de [base](../management/access-control/role-based-authorization.md) de datos pueden ver cualquier comando que se haya invocado en su base de datos.
* Otros usuarios solo pueden ver las consultas invocadas por ellos.

**Sintaxis**

`.show` `queries`

## <a name="show-running-queries"></a>.show ejecutar consultas

`.show` `running` El `queries` comando devuelve una lista de consultas que se ejecutan actualmente por el usuario, otro usuario o todos los usuarios.

**Sintaxis**

```kusto
.show running queries
```

* (1) devuelve las consultas que ejecuta actualmente el usuario que invoca (requiere acceso de lectura).

## <a name="cancel-query"></a>Consulta .cancel

El `.cancel` `query` comando inicia un intento de mejor esfuerzo para cancelar una consulta específica iniciada previamente por el mismo usuario.

* Los administradores de clúster pueden cancelar cualquier consulta en ejecución.
* Los administradores de bases de datos pueden cancelar cualquier consulta en ejecución que se haya invocado en una base de datos en la que tengan acceso de administrador.
* Otros usuarios solo pueden cancelar las consultas que iniciaron. 

**Sintaxis**

`.cancel``query` *ClientRequestId*

* *ClientRequestId* es el valor del campo ClientRequestId `string` de las consultas originales, como literal.

**Ejemplo**

```kusto
.cancel query "KE.RunQuery;8f70e9ab-958f-4955-99df-d2a288b32b2c"
```


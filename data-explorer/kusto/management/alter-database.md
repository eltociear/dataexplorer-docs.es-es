---
title: . ALTER DATABASE prettyname-Azure Explorador de datos
description: En este artículo se describe el `.alter` comando nombre descriptivo de la base de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0fc445a7d85f52d672b92163cc358d8f3a741c68
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294515"
---
# <a name="alter-database-prettyname"></a>. ALTER DATABASE prettyname

Modifica el nombre descriptivo de una base de datos.

Requiere el [permiso DatabaseAdmin](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.alter``database` *DatabaseName* `prettyname` NombreDeBaseDeDatos `'` *DatabasePrettyName*`'`

**Devolver salida**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName |String |El nombre de la base de datos.
|PrettyName |String |Nombre descriptivo de la base de datos.

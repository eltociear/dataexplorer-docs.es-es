---
title: .alter database prettyname - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el nombre bonito de la base de datos .alter en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1b362b547dc980676108ec169a0abb97f189375b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522600"
---
# <a name="alter-database-prettyname"></a>.alter base de datos prettyname

Altera el nombre bonito (amigable) de una base de datos.

Requiere [permiso DatabaseAdmin](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.alter``database` *DatabaseName* `prettyname` DatabaseName `'` *DatabasePrettyName*`'`

**Salida de retorno**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName |String |El nombre de la base de datos.
|PrettyName |String |El bonito nombre de la base de datos.


---
title: .drop database prettyname - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el nombre bonito de la base de datos .drop en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a8a74b871ddcf420f39bb45ebdf3bce36f026105
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521189"
---
# <a name="drop-database-prettyname"></a>.drop base de datos prettyname

Quita el nombre bonito (amigable) de una base de datos.
Requiere [permiso DatabaseAdmin](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.drop``database` *DatabaseName*`prettyname`

**Salida de retorno**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName |String |Nombre de la base de datos
|PrettyName |String |El nombre bonito de la base de datos (nulo después de la operación de colocación)
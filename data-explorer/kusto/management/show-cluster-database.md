---
title: 'Bases de datos de clúster .show: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen las bases de datos de clúster .show en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f354862df1bc9bef352819832125cf6f82ba0ae4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519999"
---
# <a name="show-cluster-databases"></a>Bases de datos de clúster .show

Devuelve una tabla que muestra todas las bases de datos asociadas al clúster y a la que tiene acceso el usuario que invoca el comando. Si se utilizan nombres de base de datos específicos, solo se incluirían esas bases de datos.

**Sintaxis**

`.show` `cluster` `databases` [`details` | `identity` | `policies` | `datastats`]

`.show``cluster` `,` database1`,` database2 ... `databases` `(` databaseN`)`

**Salida**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName  |String |nombre de base de datos. Los nombres de base de datos distinguen mayúsculas de minúsculas. 
|PersistentStorage  |String |El URI de almacenamiento persistente en el que se almacena la base de datos. (Este campo está vacío para bases de datos efímeras.) 
|Versión  |String |Número de versión de la base de datos. Este número se actualiza para cada operación de cambio en la base de datos (por ejemplo, agregar datos y cambiar el esquema). 
|IsCurrent  |Boolean |True si la base de datos es a la que apunta la conexión actual. 
|DatabaseAccessMode  |String |Cómo se adjunta el clúster a la base de datos. Por ejemplo, si la base de datos está asociada en modo ReadOnly, el clúster producirá un error en todas las solicitudes para modificar la base de datos de ninguna manera. 
|PrettyName |String |El nombre bonito de la base de datos.
|CurrentUserIsUnrestrictedViewer |Boolean | Especifica si el usuario actual es un visor sin restricciones en la base de datos.
|DatabaseId |Guid |Id. único de la base de datos.
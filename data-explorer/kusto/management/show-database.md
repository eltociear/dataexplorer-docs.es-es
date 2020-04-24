---
title: .show database - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe la base de datos .show en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 95d90377afd33971052dd00cd6ab3d662130824c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519880"
---
# <a name="show-database"></a>Base de datos .show

Devuelve una tabla que muestra las propiedades de la base de datos de contexto.

Para devolver una tabla en la que cada registro corresponde a una base de datos del clúster al que el usuario tiene acceso, consulte Bases de [datos .show](show-databases.md).

**Sintaxis**

`.show` `database` [`details` | `identity` | `policies` | `datastats`]

La llamada predeterminada sin ninguna opción especificada es igual a la opción 'identidad'.

**Salida para la opción 'identidad'**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName  |String |Nombre de la base de datos. Los nombres de base de datos distinguen mayúsculas de minúsculas. 
|PersistentStorage  |String |El URI de almacenamiento persistente en el que se almacena la base de datos. (Este campo está vacío para bases de datos efímeras.) 
|Versión  |String |Número de versión de la base de datos. Este número se actualiza para cada operación de cambio en la base de datos (por ejemplo, agregar datos y cambiar el esquema). 
|IsCurrent  |Boolean |True si la base de datos es a la que apunta la conexión actual. 
|DatabaseAccessMode  |String |Cómo se adjunta el clúster a la base de datos. Por ejemplo, si la base de datos está asociada en modo ReadOnly, el clúster producirá un error en todas las solicitudes para modificar la base de datos de ninguna manera. 
|PrettyName |String |El nombre bonito de la base de datos.
|CurrentUserIsUnrestrictedViewer |Boolean | Especifica si el usuario actual es un visor sin restricciones en la base de datos.
|DatabaseId |Guid |Id. único de la base de datos.

**Salida para la opción 'detalles'**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName  |String |Nombre de la base de datos. Los nombres de base de datos distinguen mayúsculas de minúsculas. 
|PersistentStorage  |String |El URI de almacenamiento persistente en el que se almacena la base de datos. (Este campo está vacío para bases de datos efímeras.) 
|Versión  |String |El número de versión de la base de datos. Este número se actualiza para cada operación de cambio en la base de datos (por ejemplo, agregar datos y cambiar el esquema). 
|IsCurrent  |Boolean |True si la base de datos es a la que apunta la conexión actual. 
|DatabaseAccessMode  |String |Cómo se adjunta el clúster a la base de datos. Por ejemplo, si la base de datos está asociada en modo ReadOnly, el clúster producirá un error en todas las solicitudes para modificar la base de datos de ninguna manera. 
|PrettyName |String |El nombre bonito de la base de datos.
|AuthorizedPrincipals |String | Colección de entidades de seguridad autorizadas de la base de datos (serializadas en formato JSON).
|RetentionPolicy |String | Política de retención de la base de datos (serializada en formato JSON).
|MergePolicy |String | La directiva Extents Merge de la base de datos (serializada en formato JSON).
|CachingPolicy |String | Política de almacenamiento en caché de la base de datos (serializada en formato JSON).
|ShardingPolicy |String | Política de partición de la base de datos (serializada en formato JSON).
|StreamingIngestionPolicy |String | Política de ingesta de streaming de la base de datos (serializada en formato JSON).
|IngestionBatchingPolicy |String | Política de procesamiento por lotes de ingesta de la base de datos (serializada en formato JSON).
|TotalSize |Real | El tamaño total de las extensiones de la base de datos.
|DatabaseId |Guid |Id. único de la base de datos.

**Salida para la opción 'políticas'**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName  |String |Nombre de la base de datos. Los nombres de base de datos distinguen mayúsculas de minúsculas. 
|PersistentStorage  |String |El URI de almacenamiento persistente en el que se almacena la base de datos. (Este campo está vacío para bases de datos efímeras.) 
|Versión  |String |El número de versión de la base de datos. Este número se actualiza para cada operación de cambio en la base de datos (por ejemplo, agregar datos y cambiar el esquema). 
|IsCurrent  |Boolean |True si la base de datos es a la que apunta la conexión actual. 
|DatabaseAccessMode  |String |Cómo se adjunta el clúster a la base de datos. Por ejemplo, si la base de datos está asociada en modo ReadOnly, el clúster producirá un error en todas las solicitudes para modificar la base de datos de ninguna manera. 
|PrettyName |String |La base de datos es un nombre bonito.
|DatabaseId |Guid |Id. único de la base de datos.
|AuthorizedPrincipals |String | Colección de entidades de seguridad autorizadas de la base de datos (serializadas en formato JSON).
|RetentionPolicy |String | Política de retención de la base de datos (serializada en formato JSON).
|MergePolicy |String | La directiva Extents Merge de la base de datos (serializada en formato JSON).
|CachingPolicy |String | Política de almacenamiento en caché de la base de datos (serializada en formato JSON).
|ShardingPolicy |String | Política de partición de la base de datos (serializada en formato JSON).
|StreamingIngestionPolicy |String | Política de ingesta de streaming de la base de datos (serializada en formato JSON).
|IngestionBatchingPolicy |String | Política de procesamiento por lotes de ingesta de la base de datos (serializada en formato JSON).

**Salida para la opción 'datastats'**

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|DatabaseName  |String |Nombre de la base de datos. Los nombres de base de datos distinguen mayúsculas de minúsculas.
|PersistentStorage  |String |El URI de almacenamiento persistente en el que se almacena la base de datos. (Este campo está vacío para bases de datos efímeras.) 
|Versión  |String |El número de versión de la base de datos. Este número se actualiza para cada operación de cambio en la base de datos (por ejemplo, agregar datos y cambiar el esquema). 
|IsCurrent  |Boolean |True si la base de datos es a la que apunta la conexión actual. 
|DatabaseAccessMode  |String |Cómo se adjunta el clúster a la base de datos. Por ejemplo, si la base de datos está asociada en modo ReadOnly, el clúster producirá un error en todas las solicitudes para modificar la base de datos de ninguna manera. 
|PrettyName |String |La base de datos es un nombre bonito.
|DatabaseId |Guid |Id. único de la base de datos.
|OriginalSize |Real | Las extensiones de la base de datos son el tamaño original total.
|ExtentSize |Real | El tamaño total de las extensiones de la base de datos (datos + índices).
|CompressedSize |Real | El tamaño total de los datos comprimidos de la base de datos en las extensiones de la base de datos.
|IndexSize |Real | El tamaño total del índice de extensiones de la base de datos.
|RowCount |long | Recuento total de filas de extensiones de la base de datos.
|HotOriginalSize |Real | Las extensiones de acceso rápido de la base de datos son el tamaño total original.
|HotExtentSize |Real | El tamaño total de las extensiones de acceso rápido de la base de datos (datos + índices).
|HotCompressedSize |Real | El tamaño total de los datos comprimidos en las extensiones de acceso rápido de la base de datos.
|HotIndexSize |Real | El tamaño total del índice de las extensiones de acceso rápido de la base de datos.
|HotRowCount |long | El recuento total de filas de las extensiones de acceso rápido de la base de datos.
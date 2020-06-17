---
title: '. Mostrar bases de datos: Azure Explorador de datos'
description: En este artículo se describe. Mostrar bases de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e5ff64db05bb85d88ed3da3347ea95cf02b0234b
ms.sourcegitcommit: 3848b8db4c3a16bda91c4a5b7b8b2e1088458a3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84818545"
---
# <a name="show-databases"></a>.show databases

Devuelve una tabla en la que cada registro corresponde a una base de datos del clúster al que el usuario tiene acceso.

Para obtener una tabla que muestra las propiedades de la base de datos de contexto, vea [`.show database`](show-database.md) .

**Sintaxis**

`.show` `databases`

**Esquema de salida**

|Nombre de la columna       |Tipo de columna|Descripción                                                                  |
|------------------|-----------|-----------------------------------------------------------------------------|
|DatabaseName      |`string`   |El nombre de la base de datos asociada al clúster                    |
|PersistentStorage |`string`   |El almacenamiento persistente "raíz" de la base de datos. Exclusivamente para uso interno.          |
|Versión           |`string`   |La versión de la base de datos. Exclusivamente para uso interno.                       |
|IsCurrent         |`bool`     |Si esta base de datos es el contexto de la base de datos de la solicitud                    |
|DatabaseAccessMode|`string`   |Uno de `ReadWrite` , `ReadOnly` , `ReadOnlyFollowing` o`ReadWriteEphemeral`    |
|PrettyName        |`string`   |Nombre descriptivo de la base de datos, si existe.                        |
|ReservedSlot1     |`bool`     |Reservado. Exclusivamente para uso interno.              |
|DatabaseId        |`guid`     |Un identificador único global para la base de datos. Exclusivamente para uso interno.          |

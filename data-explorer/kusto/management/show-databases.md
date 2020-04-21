---
title: 'Bases de datos .show: Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las bases de datos .show en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1827c3ea20d984c6846ef9feb603398020efe275
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610195"
---
# <a name="show-databases"></a>Bases de datos .show

Devuelve una tabla en la que cada registro corresponde a una base de datos del clúster al que el usuario tiene acceso.

Para ver devolver una tabla que muestra las propiedades de la base de datos de contexto, vea [.show database](show-database.md).

**Sintaxis**

`.show` `databases`

**Esquema de salida**

|Nombre de la columna       |Tipo de columna|Descripción                                                                  |
|------------------|-----------|-----------------------------------------------------------------------------|
|DatabaseName      |`string`   |El nombre de la base de datos asociada al clúster.                         |
|PersistentStorage |`string`   |El almacenamiento persistente "raíz" de la base de datos. (Solo para uso interno.)      |
|Versión           |`string`   |La versión de la base de datos. (Solo para uso interno.)                        |
|IsCurrent         |`bool`     |Si esta base de datos es el contexto de base de datos de la solicitud.                |
|DatabaseAccessMode|`string`   |Uno `ReadWrite`de `ReadOnly` `ReadOnlyFollowing`, `ReadWriteEphemeral`, , o .|
|PrettyName        |`string`   |El nombre bonito de la base de datos, si existe.                                    |
|ReservedSlot1     |`bool`     |Reservado. (Solo para uso interno.)                                           |
|DatabaseId        |`guid`     |Identificador único global para la base de datos. (Solo para uso interno.)      |

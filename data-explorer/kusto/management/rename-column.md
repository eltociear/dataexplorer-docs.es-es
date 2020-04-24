---
title: 'columna de cambio de nombre: Explorador de Azure Data Explorer ? Microsoft Docs'
description: En este artículo se describe la columna de cambio de nombre en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 02e692e01bb8eb0abd49c9673b000b722734db90
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520509"
---
# <a name="rename-column"></a>cambiar el nombre de la columna

Cambia el nombre de una columna de tabla existente.
Para cambiar el nombre de varias columnas, consulte [a continuación](#rename-columns).

**Sintaxis**

`.rename``column` [*DatabaseName* `.`] *NombreDeTabla* `.` *ColumnExistingName* `to` *ColumnNewName*

Donde *DatabaseName*, *TableName*, *ColumnExistingName*y *ColumnNewName* son los nombres de las entidades respectivas y siguen las reglas de nomenclatura del [identificador.](../query/schema-entities/entity-names.md)

## <a name="rename-columns"></a>cambiar el nombre de las columnas

Cambia los nombres de varias columnas existentes en la misma tabla.

**Sintaxis**

`.rename``columns` *Col1* `=` [*DatabaseName* `.` [*TableName* `.` *Col2*]] `,` ...

El comando se puede utilizar para intercambiar los nombres de dos columnas (cada una cambia de nombre como el nombre del otro.)
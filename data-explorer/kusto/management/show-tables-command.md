---
title: 'Tablas .show: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las tablas .show en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: a8faf307a241d1ba0f73436d9503a56c9078e471
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519642"
---
# <a name="show-tables"></a>Tablas .show

Devuelve un conjunto que contiene la tabla especificada o todas las tablas de la base de datos.

Requiere [permiso Visor](../management/access-control/role-based-authorization.md)de base de datos .

```
.show tables
.show tables (T1, ..., Tn)
```

**Salida**

|Parámetro de salida |Tipo |Descripción
|---|---|---
|TableName  |String |Nombre de la tabla.
|DatabaseName  |String |La base de datos a la que pertenece la tabla.
|Carpeta |String |La carpeta de la tabla.
|DocString |String |Una cadena que documenta la tabla.

**Ejemplo de salida**

|Nombre de la tabla |Nombre de la base de datos |Carpeta | DocString
|---|---|---|---
|Table1 |DB1 |Registros |Contiene registros de servicios
|Table2 |DB1 | Notificación |
|Tabla3 |DB1 | | Información ampliada |
|Tabla4 |DB2 | Métricas| Contiene información de rendimiento de los servicios
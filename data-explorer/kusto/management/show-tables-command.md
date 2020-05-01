---
title: '. Mostrar tablas: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Mostrar tablas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 3da4f705d3182c52d06c7767a12d9be15a219e5c
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618333"
---
# <a name="show-tables"></a>. Mostrar tablas

Devuelve un conjunto que contiene la tabla especificada o todas las tablas de la base de datos.

Requiere el [permiso visor de bases de datos](../management/access-control/role-based-authorization.md).

```kusto
.show tables
.show tables (T1, ..., Tn)
```

**Salida**

|Parámetro de salida |Tipo |Descripción
|---|---|---
|TableName  |String |Nombre de la tabla.
|DatabaseName  |String |La base de datos a la que pertenece la tabla.
|Carpeta |String |La carpeta de la tabla.
|DocString |String |Cadena que documenta la tabla.

**Ejemplo de salida**

|Nombre de la tabla |Nombre de la base de datos |Carpeta | DocString
|---|---|---|---
|Table1 |DB1 |Registros |Contiene registros de servicios
|Table2 |DB1 | Notificación |
|Tabla3 |DB1 | | Información ampliada |
|Table4 |DB2 | Métricas| Contiene información de rendimiento de servicios

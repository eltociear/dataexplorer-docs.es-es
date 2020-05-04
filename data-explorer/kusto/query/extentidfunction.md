---
title: extent_id ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe extent_id () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 300f7961fd11b433ef4e420d5a20b9ad9150b269
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737630"
---
# <a name="extent_id"></a>extent_id()

::: zone pivot="azuredataexplorer"

Devuelve un identificador único que identifica la partición de datos ("extensión") en la que reside el registro actual. 

Al aplicar esta función a los datos calculados que no se adjuntan a una partición de datos, se devuelve un GUID vacío (todo ceros).

**Sintaxis**

`extent_id()`

**Devuelve**

Un valor de tipo `guid` que identifica la partición de datos del registro actual o un GUID vacío (todo ceros).

**Ejemplo**

En el ejemplo siguiente se muestra cómo obtener una lista de todas las particiones de datos que tienen registros desde hace una hora con un valor específico para `ActivityId`la columna. Muestra que algunos operadores de consulta (aquí, el `where` operador, pero esto también se cumple para `extend` y `project`) conservan la información sobre la partición de datos que hospeda el registro.

```kusto
T
| where Timestamp > ago(1h)
| where ActivityId == 'dd0595d4-183e-494e-b88e-54c52fe90e5a'
| extend eid=extent_id()
| summarize by eid
```

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

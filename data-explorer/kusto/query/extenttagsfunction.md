---
title: extent_tags() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe extent_tags() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: d8af7e51c5e2efb16763541db05e9ccc7e2cb95f
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765432"
---
# <a name="extent_tags"></a>extent_tags()

::: zone pivot="azuredataexplorer"

Devuelve una matriz dinámica con las [etiquetas](../management/extents-overview.md#extent-tagging) de la partición de datos ("extensión") en la que reside el registro actual. 

La aplicación de esta función a los datos calculados que no están asociados a una partición de datos devuelve un valor vacío.

**Sintaxis**

`extent_tags()`

**Devuelve**

Valor de `dynamic` tipo que es una matriz que contiene las etiquetas de extensión del registro actual o un valor vacío.

**Ejemplos**

En el ejemplo siguiente se muestra cómo obtener una lista de las etiquetas de todas las `ActivityId`particiones de datos que tienen registros de hace una hora con un valor específico para la columna. Demuestra que algunos operadores de `where` consulta (aquí, el `extend` `project`operador, pero esto también es cierto para y ) conservan la información sobre la partición de datos que hospeda el registro.

```kusto
T
| where Timestamp > ago(1h)
| where ActivityId == 'dd0595d4-183e-494e-b88e-54c52fe90e5a'
| extend tags = extent_tags()
| summarize by tostring(tags)
```

En el ejemplo siguiente se muestra cómo obtener un recuento de todos los registros `MyTag` de la última hora, que se `drop-by:MyOtherTag`almacenan en extensiones que están etiquetadas con la etiqueta (y potencialmente otras etiquetas), pero no etiquetadas con la etiqueta .

```kusto
T
| where Timestamp > ago(1h)
| extend Tags = extent_tags()
| where Tags has_cs 'MyTag' and Tags !has_cs 'drop-by:MyOtherTag'
| count
```

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

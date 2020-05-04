---
title: extent_tags ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe extent_tags () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 146ab59c1c0cbcb86bfae94fa26c09f5afa0f216
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737596"
---
# <a name="extent_tags"></a>extent_tags()

::: zone pivot="azuredataexplorer"

Devuelve una matriz dinámica con las [etiquetas](../management/extents-overview.md#extent-tagging) de la partición de datos ("extent") en la que reside el registro actual. 

Al aplicar esta función a los datos calculados que no se adjuntan a una partición de datos, se devuelve un valor vacío.

**Sintaxis**

`extent_tags()`

**Devuelve**

Un valor de tipo `dynamic` que es una matriz que contiene las etiquetas de extensión del registro actual o un valor vacío.

**Ejemplos**

En el ejemplo siguiente se muestra cómo obtener una lista de las etiquetas de todas las particiones de datos que tienen registros desde hace una hora con un valor específico `ActivityId`para la columna. Muestra que algunos operadores de consulta (aquí, el `where` operador, pero esto también se cumple para `extend` y `project`) conservan la información sobre la partición de datos que hospeda el registro.

```kusto
T
| where Timestamp > ago(1h)
| where ActivityId == 'dd0595d4-183e-494e-b88e-54c52fe90e5a'
| extend tags = extent_tags()
| summarize by tostring(tags)
```

En el ejemplo siguiente se muestra cómo obtener un recuento de todos los registros de la última hora, que se almacenan en extensiones que se etiquetan con la etiqueta `MyTag` (y potencialmente otras etiquetas), pero no `drop-by:MyOtherTag`se etiquetan con la etiqueta.

```kusto
T
| where Timestamp > ago(1h)
| extend Tags = extent_tags()
| where Tags has_cs 'MyTag' and Tags !has_cs 'drop-by:MyOtherTag'
| count
```

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

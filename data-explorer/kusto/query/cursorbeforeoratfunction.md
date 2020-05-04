---
title: cursor_before_or_at ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe cursor_before_or_at () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: c053cd307f8cff8ad00eff0a4224ebbea2808c6c
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737681"
---
# <a name="cursor_before_or_at"></a>cursor_before_or_at()

::: zone pivot="azuredataexplorer"

Un predicado sobre los registros de una tabla para comparar su tiempo de ingesta con un cursor de base de datos.

**Sintaxis**

`cursor_before_or_at``(` *RHS*`)`

**Argumentos**

* *RHS*: un literal de cadena vacío o un valor de cursor de base de datos válido.

**Devuelve**

Un valor escalar de `bool` tipo que indica si el registro se ingesta antes o en el cursor de base`true`de datos *RHS* (`false`) o no ().

**Notas**

Vea [cursores de base de datos](../management/databasecursor.md) para obtener más detalles sobre los cursores de base de datos.

Esta función solo se puede invocar en registros de una tabla que tenga habilitada la [Directiva IngestionTime](../management/ingestiontimepolicy.md) .

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

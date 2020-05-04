---
title: cursor_after ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe cursor_after () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 9fab1ec936e950368667fc3afb133dcd952e44b5
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737698"
---
# <a name="cursor_after"></a>cursor_after()

::: zone pivot="azuredataexplorer"

Un predicado sobre los registros de una tabla para comparar su tiempo de ingesta con un cursor de base de datos.

**Sintaxis**

`cursor_after``(` *RHS*`)`

**Argumentos**

* *RHS*: un literal de cadena vacío o un valor de cursor de base de datos válido.

**Devuelve**

Un valor escalar de `bool` tipo que indica si el registro se ingesta después del cursor de la`true`base de datos *RHS* () o no (`false`).

**Notas**

Vea [cursores de base de datos](../management/databasecursor.md) para obtener más detalles sobre los cursores de base de datos.

Esta función solo se puede invocar en registros de una tabla que tenga habilitada la [Directiva IngestionTime](../management/ingestiontimepolicy.md) .

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

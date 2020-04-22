---
title: cursor_before_or_at() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe cursor_before_or_at() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 4d1752c69a6f3515b94c4050cef8f518ff58a235
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765971"
---
# <a name="cursor_before_or_at"></a>cursor_before_or_at()

::: zone pivot="azuredataexplorer"

Un predicado sobre los registros de una tabla para comparar su tiempo de ingesta con un cursor de base de datos.

**Sintaxis**

`cursor_before_or_at``(` *RHS*`)`

**Argumentos**

* *RHS*: un literal de cadena vacío o un valor de cursor de base de datos válido.

**Devuelve**

Valor escalar de tipo `bool` que indica si el registro se ingirió antes`true`o en`false`el cursor de base de datos *RHS* ( ) o no ( ).

**Notas**

Consulte [cursores](../management/databasecursor.md) de base de datos para obtener más información sobre los cursores de base de datos.

Esta función solo se puede invocar en registros de una tabla que tenga habilitada la [directiva IngestionTime.](../management/ingestiontimepolicy.md)

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

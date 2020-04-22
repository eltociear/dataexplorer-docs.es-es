---
title: cursor_after() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe cursor_after() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 39ec32322b74b55182522e4bbb04aa0c3830d8d2
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766014"
---
# <a name="cursor_after"></a>cursor_after()

::: zone pivot="azuredataexplorer"

Un predicado sobre los registros de una tabla para comparar su tiempo de ingesta con un cursor de base de datos.

**Sintaxis**

`cursor_after``(` *RHS*`)`

**Argumentos**

* *RHS*: un literal de cadena vacío o un valor de cursor de base de datos válido.

**Devuelve**

Valor escalar de tipo `bool` que indica si el registro se ingirió`true`después del`false`cursor de base de datos *RHS* ( ) o no ( ).

**Notas**

Consulte [cursores](../management/databasecursor.md) de base de datos para obtener más información sobre los cursores de base de datos.

Esta función solo se puede invocar en registros de una tabla que tenga habilitada la [directiva IngestionTime.](../management/ingestiontimepolicy.md)

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

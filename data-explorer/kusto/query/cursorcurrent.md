---
title: cursor_current (), current_cursor ()-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe cursor_current (), current_cursor () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: d9a9526fd846523510e8555c04ff3345d9008348
ms.sourcegitcommit: fbe298e88542c0dcea0f491bb53ac427f850f729
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/24/2020
ms.locfileid: "82030470"
---
# <a name="cursor_current-current_cursor"></a>cursor_current(), current_cursor()

::: zone pivot="azuredataexplorer"

Recupera el valor actual del cursor de la base de datos en el ámbito. (Los nombres `cursor_current` y `current_cursor` son sinónimos).

**Sintaxis**

`cursor_current()`

**Devuelve**

Devuelve un valor único de tipo `string` que codifica el valor actual del cursor de la base de datos en el ámbito.

**Notas**

Vea [cursores de base de datos](../management/databasecursor.md) para obtener más detalles sobre los cursores de base de datos.

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

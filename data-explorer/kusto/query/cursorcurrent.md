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
ms.openlocfilehash: 5291fa664dc4736179d7f20984eacfd44efd5888
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737664"
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

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

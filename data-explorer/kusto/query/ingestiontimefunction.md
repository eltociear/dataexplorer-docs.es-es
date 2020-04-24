---
title: ingestion_time ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe ingestion_time () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 2474b8751be5cba2270bcbd2936c76b91f5f3ba0
ms.sourcegitcommit: fbe298e88542c0dcea0f491bb53ac427f850f729
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/24/2020
ms.locfileid: "82030046"
---
# <a name="ingestion_time"></a>ingestion_time()

Recupera la columna oculta `$IngestionTime` `datetime` del registro o null.
La `$IngestionTime` columna se define automáticamente cuando el

::: zone pivot="azuredataexplorer"

La [Directiva IngestionTime](../management/ingestiontimepolicy.md) está establecida (habilitada).

::: zone-end

::: zone pivot="azuremonitor"

La Directiva IngestionTime está establecida (habilitada).

::: zone-end

Si la tabla no tiene definida esta Directiva, se devuelve un valor null.

Esta función debe usarse en el contexto de una tabla real para devolver los datos pertinentes. Por ejemplo, devolverá null para todos los registros si se invoca después de un `summarize` operador.

**Sintaxis**

 `ingestion_time()`

**Devuelve**

`datetime` Valor que especifica el tiempo aproximado de ingesta en una tabla.

**Ejemplo**

```kusto
T 
| extend ingestionTime = ingestion_time() | top 10 by ingestionTime
```

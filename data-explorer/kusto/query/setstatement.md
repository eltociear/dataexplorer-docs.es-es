---
title: 'Establecer instrucción: Explorador de azure Data Explorer ? Microsoft Docs'
description: En este artículo se describe la instrucción Set en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 8cb9c1d72f1b2e8e4bfbbd28d67c04295c9ccf5b
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765575"
---
# <a name="set-statement"></a>Instrucción Set

::: zone pivot="azuredataexplorer"

La `set` instrucción se utiliza para establecer una opción de consulta durante la duración de la consulta.
Las opciones de consulta controlan cómo se ejecuta una consulta y devuelve los resultados. Pueden ser indicadores booleanos (desactivados de forma predeterminada) o tener un valor entero. Una consulta puede contener cero, una o más instrucciones set. Las instrucciones Set solo afectan a las instrucciones de expresión tabular que las rastrean en el orden del programa.

* Las opciones de consulta también se pueden `ClientRequestProperties` habilitar mediante programación estableciéndolas en el objeto. Consulte [aquí](../api/netfx/request-properties.md).
  
* Las opciones de consulta no forman parte formalmente del lenguaje Kusto y pueden modificarse sin considerarse como un cambio de idioma de última hora.

**Sintaxis**

`set`*OptionName* `=` [ *OptionValue*]

**Ejemplo**

```kusto
set querytrace;
Events | take 100
```

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

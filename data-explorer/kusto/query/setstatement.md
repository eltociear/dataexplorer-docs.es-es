---
title: 'Instrucción set: Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe la instrucción set en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 028bfb5a2d0ddf25f65cd16bca2c498d9dcb7059
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737868"
---
# <a name="set-statement"></a>Instrucción Set

::: zone pivot="azuredataexplorer"

La `set` instrucción se utiliza para establecer una opción de consulta para la duración de la consulta.
Las opciones de consulta controlan cómo se ejecuta una consulta y devuelve los resultados. Pueden ser marcas booleanas (desactivadas de forma predeterminada) o tener un valor entero. Una consulta puede contener cero, una o más instrucciones set. Las instrucciones SET solo afectan a las instrucciones de expresión tabular que las finalizan en el orden del programa.

* Las opciones de consulta también se pueden habilitar mediante programación si se establecen `ClientRequestProperties` en el objeto. Consulte [aquí](../api/netfx/request-properties.md).
  
* Las opciones de consulta no son formalmente una parte del lenguaje Kusto y se pueden modificar sin tener en cuenta el cambio de lenguaje problemático.

**Sintaxis**

`set`*OptionName* [`=` *OptionValue*]

**Ejemplo**

```kusto
set querytrace;
Events | take 100
```

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

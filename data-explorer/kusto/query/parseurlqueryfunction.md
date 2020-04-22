---
title: parse_urlquery() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe parse_urlquery() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3f00fefcd6245528d7ae50d6046d97289a92317d
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744617"
---
# <a name="parse_urlquery"></a>parse_urlquery()

Devuelve `dynamic` un objeto que contiene los parámetros Query.

**Sintaxis**

`parse_urlquery(`*Consulta*`)`

**Argumentos**

* *consulta*: Una cadena representa una consulta url.

**Devuelve**

Objeto de tipo [dynamic](./scalar-data-types/dynamic.md) que incluye los parámetros de consulta.

**Ejemplo**

```kusto
parse_urlquery("k1=v1&k2=v2&k3=v3")
```

resultará:

```kusto
 {
    "Query Parameters":"{"k1":"v1", "k2":"v2", "k3":"v3"}",
 }
```

**Notas**

* El formato de entrada debe seguir los estándares de consulta de URL (clave-valor& ...)
 
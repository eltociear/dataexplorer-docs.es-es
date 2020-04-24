---
title: parse_url() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_url() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: dfc093964ce5b91acc01f798f8f62651266ab153
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511499"
---
# <a name="parse_url"></a>parse_url()

Analiza una dirección `string` URL `dynamic` absoluta y devuelve un objeto que contiene partes de URL.


**Sintaxis**

`parse_url(`*Url*`)`

**Argumentos**

* *url*: Una cadena representa una dirección URL o la parte de consulta de la dirección URL.

**Devuelve**

Objeto de tipo [dinámico](./scalar-data-types/dynamic.md) que incluía los componentes de URL: Scheme, Host, Port, Path, Username, Password, Query Parameters, Fragment.

**Ejemplo**

```kusto
T | extend Result = parse_url("scheme://username:password@host:1234/this/is/a/path?k1=v1&k2=v2#fragment")
```

resultará

```
 {
    "Scheme":"scheme",
    "Host":"host",
    "Port":"1234",
    "Path":"this/is/a/path",
    "Username":"username",
    "Password":"password",
    "Query Parameters":"{"k1":"v1", "k2":"v2"}",
    "Fragment":"fragment"
 }
```
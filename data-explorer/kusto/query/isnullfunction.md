---
title: IsNull ()-Azure Explorador de datos
description: En este artículo se describe IsNull () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4c57c7aba2bff2dfaecfa72b20ab76cc84ed17d6
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550595"
---
# <a name="isnull"></a>isnull()

Evalúa su único argumento y devuelve un `bool` valor que indica si el argumento se evalúa como un valor null.

```kusto
isnull(parse_json("")) == true
```

**Sintaxis**

`isnull(`*Argumento*`)`

**Devuelve**

True o false, dependiendo de si el valor es null o no.

**Notas**

* `string`los valores no pueden ser null. Use [IsEmpty](./isemptyfunction.md) para determinar si un valor de tipo `string` está vacío o no.

|x                |`isnull(x)`|
|-----------------|-----------|
|`""`             |`false`    |
|`"x"`            |`false`    |
|`parse_json("")`  |`true`     |
|`parse_json("[]")`|`false`    |
|`parse_json("{}")`|`false`    |

**Ejemplo**

```kusto
T | where isnull(PossiblyNull) | count
```
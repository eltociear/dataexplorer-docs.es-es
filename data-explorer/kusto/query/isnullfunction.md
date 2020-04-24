---
title: isnull() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe isnull() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e26dca661ceac1ad209358b24b3f8d497a5c3049
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513420"
---
# <a name="isnull"></a>isnull()

Evalúa su único argumento `bool` y devuelve un valor que indica si el argumento se evalúa como un valor nulo.

```kusto
isnull(parse_json("")) == true
```

**Sintaxis**

`isnull(`*Expr*`)`

**Devuelve**

Verdadero o falso dependiendo de si el valor es null o not null.

**Notas**

* `string`los valores no pueden ser nulos. Utilice [isempty](./isemptyfunction.md) para determinar si `string` un valor de type está vacío o no.

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
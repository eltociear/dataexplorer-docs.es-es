---
title: series_not_equals() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_not_equals() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: ce9c695ececf1ac9f1fbe783ebe0fa35986f3d0f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508184"
---
# <a name="series_not_equals"></a>series_not_equals()

Calcula la operación lógica no`!=`es igual a ( ) por elementos de dos entradas de serie numérica.

**Sintaxis**

`series_not_equals (`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, Series2*: Introduzca matrices numéricas para que se comparen en cuanto a elementos. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de booleanos que contiene la operación lógica no igual a elemento calculada entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices `null` de diferentes tamaños) produce un valor de elemento.

**Ejemplo**

```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_not_equals_s2 = series_not_equals(s1, s2)
```

|s1|s2|s1_not_equals_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[true,false,true]|

**Vea también**

Para comparaciones de estadísticas de series completas, véase:
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
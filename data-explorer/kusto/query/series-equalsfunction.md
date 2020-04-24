---
title: series_equals() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_equals() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 267e9db8dd2980016f4022ce6358569f30aacb6f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508813"
---
# <a name="series_equals"></a>series_equals()

Calcula la operación lógica`==`igual a elemento sin ( ) de dos entradas de serie numérica.

**Sintaxis**

`series_equals (`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, Series2*: Introduzca matrices numéricas para que se comparen en cuanto a elementos. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de booleanos que contiene la operación lógica igual calculada en cuanto a elementos entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices `null` de diferentes tamaños) produce un valor de elemento.

**Ejemplo**

```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_equals_s2 = series_equals(s1, s2)
```

|s1|s2|s1_equals_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[falso,verdadero,falso]|

**Vea también**

Para comparaciones de estadísticas de series completas, véase:
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
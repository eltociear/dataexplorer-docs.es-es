---
title: range() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe range() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 86558591e6312edd218230cda19a4afc17a13b27
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744586"
---
# <a name="range"></a>range()

Genera una matriz dinámica que contiene una serie de valores igualmente espaciados.

**Sintaxis**

`range(`*start* `,` *stop*stop`,` [ *step*]`)` 

**Argumentos**

* *start*: el valor del primer elemento de la matriz resultante. 
* *stop*: el valor del último elemento de la matriz resultante, o el valor mínimo que es mayor que el último elemento de la matriz resultante y dentro de un múltiplo entero de *paso* desde *el inicio*.
* *paso*: La diferencia entre dos elementos consecutivos de la matriz. El valor predeterminado `1` para el `1h` `timespan` *paso* es para numérico y para o`datetime`

**Ejemplos**

El siguiente ejemplo devuelve `[1, 4, 7]`:

```kusto
T | extend r = range(1, 8, 3)
```

El ejemplo siguiente devuelve una matriz que contiene todos los días del año 2015:

```kusto
T | extend r = range(datetime(2015-01-01), datetime(2015-12-31), 1d)
```

El siguiente ejemplo devuelve `[1,2,3]`:

```kusto
range(1, 3)
```

El siguiente ejemplo devuelve `["01:00:00","02:00:00","03:00:00","04:00:00","05:00:00"]`:

```kusto
range(1h, 5h)
```

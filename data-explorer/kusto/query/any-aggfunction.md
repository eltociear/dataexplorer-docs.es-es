---
title: any() (función de agregación) - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe any() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0726b185c22cd84c93e28601a823a35d9685d96d
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663942"
---
# <a name="any-aggregation-function"></a>any() (función de agregación)

Elige arbitrariamente un registro para cada grupo en un operador de [resumen](summarizeoperator.md)y devuelve el valor de una o más expresiones sobre cada uno de estos registros.

**Sintaxis**

`summarize``any` *Expr* `,` *Expr2* ( Expr [ Expr2 ...]) `(` `*``)`

**Argumentos**

* *Expr*: Una expresión sobre cada registro seleccionado de la entrada que se va a devolver.
* *Expr2* .. *ExprN*: Expresiones adicionales.

**Devuelve**

La `any` función de agregación devuelve los valores de las expresiones calculadas para cada uno de los registros, seleccionados aleatoriamente de cada grupo del operador de resumen.

Si `*` se proporciona el argumento, la función se comporta como si las expresiones fueran todas las columnas de la entrada para el operador de resumen que franjan las columnas group-by, si las hubiera.

**Comentarios:**

Esta función es útil cuando desea obtener un valor de muestra de una o más columnas por valor de la clave de grupo compuesto.

Cuando la función se proporciona con una sola referencia de columna, intentará devolver un valor no nulo o no vacío, si dicho valor está presente.

Como resultado de la naturaleza aleatoria de esta función, usarlo `summarize` varias veces en una sola aplicación del operador no equivale a usarlo una sola vez con varias expresiones. El primero puede hacer que cada aplicación seleccione un registro diferente, mientras que el segundo garantiza que todos los valores se calculan en un único registro (por grupo distinto).

**Ejemplos**

Mostrar continente aleatorio:

```kusto
Continents | summarize any(Continent)
```

:::image type="content" source="images/aggregations/any1.png" alt-text="Cualquier 1":::

Mostrar todos los detalles de un registro aleatorio:

```kusto
Continents | summarize any(*)
```

:::image type="content" source="images/aggregations/any2.png" alt-text="Cualquier 2":::

Mostrar todos los detalles de cada continente aleatorio:

```kusto
Continents | summarize any(*) by Continent
```

:::image type="content" source="images/aggregations/any3.png" alt-text="Cualquier 3":::

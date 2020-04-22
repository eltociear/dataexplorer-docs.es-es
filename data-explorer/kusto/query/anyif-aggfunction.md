---
title: anyif() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe anyif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5e19332cf464fcad1715f4feddfe7a2c5765bb0d
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663996"
---
# <a name="anyif-aggregation-function"></a>anyif() (función de agregación)

Elige arbitrariamente un registro para cada grupo en un operador de [resumen](summarizeoperator.md) para el que el predicado es true y devuelve el valor de una expresión sobre cada uno de estos registros.

**Sintaxis**

`summarize`Expr , *Predicado* )' *Expr* `anyif` `(`

**Argumentos**

* *Expr*: Una expresión sobre cada registro seleccionado de la entrada que se va a devolver.
* *Predicado*: Predicado para indicar qué registros se pueden considerar para la evaluación.

**Devuelve**

La `anyif` función de agregación devuelve el valor de la expresión calculada para cada uno de los registros seleccionados aleatoriamente de cada grupo del operador de resumen. Solo se pueden seleccionar los registros para los que *Predicate* devuelve true (si el predicado no devuelve true, se genera un valor nulo).

**Comentarios:**

Esta función es útil cuando desea obtener un valor de muestra de una columna por valor de la clave de grupo compuesto, sujeto a que algún predicado sea true.

La función intenta devolver un valor no nulo o no vacío, si dicho valor está presente.

**Ejemplos**

Mostrar continente aleatorio que tiene una población de 300 millones a 600 millones:

```kusto
Continents | summarize anyif(Continent, Population between (300000000 .. 600000000))
```

:::image type="content" source="images/aggregations/any1.png" alt-text="Cualquier 1":::

---
title: 'anyif () (función de agregación): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe anyif () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 54431e2d088f60fa8ea2a56bffea9faa374faeda
ms.sourcegitcommit: aaada224e2f8824b51e167ddb6ff0bab92e5485f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2020
ms.locfileid: "84626663"
---
# <a name="anyif-aggregation-function"></a>anyif () (función de agregación)

Selecciona arbitrariamente un registro para cada grupo en un [operador de Resumen](summarizeoperator.md), para el que el predicado es "true". La función devuelve el valor de una expresión en cada registro de este tipo.

**Sintaxis**

`summarize``anyif` `(` *Expr*, *predicado*`)`

**Argumentos**

* *Expr*: expresión sobre cada registro seleccionado en la entrada que se va a devolver.
* *Predicate*: predicado para indicar qué registros se pueden considerar para su evaluación.

**Devuelve**

La `anyif` función de agregación devuelve el valor de la expresión calculada para cada uno de los registros seleccionados aleatoriamente de cada grupo del operador de resumen. Solo se pueden seleccionar los registros para los que el *predicado* devuelve "true". Si el predicado no devuelve "true", se genera un valor null.

**Comentarios:**

Esta función resulta útil cuando se desea obtener un valor de ejemplo de una columna por cada valor de la clave del grupo compuesto, sujeto a algún predicado que sea "true".

La función intenta devolver un valor que no es null o no está vacío, si ese valor está presente.

**Ejemplos**

Muestra un continente aleatorio con un rellenado de 300 a 600 millones.

```kusto
Continents | summarize anyif(Continent, Population between (300000000 .. 600000000))
```

:::image type="content" source="images/aggfunction/any1.png" alt-text="Cualquier 1":::

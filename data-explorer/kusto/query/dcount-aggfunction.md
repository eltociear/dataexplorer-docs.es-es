---
title: dcount() (función de agregación) - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe dcount() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6f1df8c93d21b73be3753468708a119177d4d602
ms.sourcegitcommit: 29018b3db4ea7d015b1afa65d49ecf918cdff3d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "82030430"
---
# <a name="dcount-aggregation-function"></a>dcount() (función de agregación)

Devuelve una estimación para el número de valores distintos tomados por una expresión escalar en el grupo de resumen.

**Sintaxis**

... `|` `,` *Accuracy*`)` *Expr* Expr [ Precisión ] ... `summarize` `dcount` `(`

**Argumentos**

* *Expr*: Una expresión escalar cuyos valores distintos se deben contar.
* *Precisión*: Un `int` literal opcional que define la precisión de estimación solicitada. Consulte a continuación los valores admitidos. Si no se especifica, `1` se utiliza el valor predeterminado.

**Devuelve**

Devuelve una estimación del número de valores distintivos de *Expr* en el grupo.

**Ejemplo**

```kusto
PageViewLog | summarize countries=dcount(country) by continent
```

:::image type="content" source="images/dcount-aggfunction/dcount.png" alt-text="D conteo":::

**Notas**

La `dcount()` función de agregación es principalmente útil para estimar la cardinalidad de conjuntos enormes. Opera el rendimiento por precisión, y por lo tanto puede devolver un resultado que varía entre las ejecuciones (orden de entradas puede tener un efecto en su salida).

Para obtener un recuento preciso `V` de `G`valores distintos de agrupados por:

```kusto
T | summarize by V, G | summarize count() by G
```

Este cálculo requerirá mucha memoria interna `V` ya que los valores `G`distintos de se multiplican por el número de valores distintos de ; Por lo tanto, puede dar lugar a errores de memoria o tiempos de ejecución grandes. `dcount()`proporciona una alternativa rápida y fiable:

```kusto
T | summarize dcount(B) by G | count
```

## <a name="estimation-accuracy"></a>Precisión de estimación

La `dcount()` función de agregado utiliza una variante del [algoritmo HyperLogLog (HLL),](https://en.wikipedia.org/wiki/HyperLogLog)que realiza una estimación estocástica de la cardinalidad del conjunto. El algoritmo proporciona un "botón" que se puede utilizar para equilibrar la precisión y el tiempo de ejecución / tamaño de memoria:

|Precisión|Error (%)|Recuento de entradas   |
|--------|---------|--------------|
|       0|      1.6|2<sup>12</sup>|
|       1|      0.8|2<sup>14</sup>|
|       2|      0,4|2<sup>16</sup>|
|       3|     0,28|2<sup>17</sup>|
|       4|      0,2|2<sup>18</sup>|

> [!NOTE]
> La columna "recuento de entradas" es el número de contadores de 1 byte en la implementación de HLL.

El algoritmo incluye algunas disposiciones para hacer un recuento perfecto (error cero) si la cardinalidad `1`del conjunto es lo suficientemente pequeña `2`(1000 valores cuando el nivel de precisión es , y 8000 valores si el nivel de precisión es ).

El límite de error es probabilístico, no un límite teórico. El valor es la desviación estándar de la distribución de errores (el sigma), y el 99,7% de las estimaciones tendrán un error relativo de menos de 3 veces sigma.

A continuación se muestra la función de distribución de probabilidad del error de estimación relativa (en porcentajes) para todos los valores de precisión admitidos:

:::image type="content" border="false" source="images/dcount-aggfunction/hll-error-distribution.png" alt-text="distribución de errores hll":::

---
title: dcountif() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe dcountif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1cc3c17474db835f381cac32d6107acb312c3d11
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516055"
---
# <a name="dcountif-aggregation-function"></a>dcountif() (función de agregación)

Devuelve una estimación del número de valores distintos de `true` *Expr* de filas para las que *Predicate* se evalúa como . 

* Solo se puede utilizar en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Lea acerca de la precisión de [estimación](dcount-aggfunction.md#estimation-accuracy).

**Sintaxis**

resumir `dcountif(` *Expr*,`,` *Predicado*, [ *Precisión*]`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.
* *Predicado*: Expresión que se utilizará para filtrar filas.
* *Accuracy*, si se especifica, controla el equilibrio entre velocidad y precisión.
    * `0` = el cálculo menos preciso y más rápido. Error del 1,6%
    * `1`• el valor por defecto, que equilibra la precisión y el tiempo de cálculo; alrededor de 0.8% error.
    * `2`• cálculo preciso y lento; alrededor de 0.4% error.
    * `3`• cálculo extra preciso y lento; alrededor de 0.28% de error.
    * `4`• cálculo súper preciso y más lento; alrededor de 0.2% de error.
    
**Devuelve**

Devuelve una estimación del número de valores distintos de `true` *Expr* de filas para las que *Predicate* se evalúa en el grupo. 

**Ejemplo**

```kusto
PageViewLog | summarize countries=dcountif(country, country startswith "United") by continent
```

**Consejo: Error de desplazamiento**

`dcountif()`un error único en los casos de borde donde todas las filas pasan, o ninguna de las filas pasa, la `Predicate` expresión
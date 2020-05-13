---
title: 'ejemplo: operador DISTINCT-Azure Explorador de datos'
description: En este artículo se describe el operador Sample-DISTINCT de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5303801b983b326310065ea2a6ce6ded7d098001
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373008"
---
# <a name="sample-distinct-operator"></a>Operador sample-distinct

Devuelve una única columna que contiene hasta el número especificado de valores distintos de la columna solicitada. 

el tipo predeterminado (y actualmente solo) del operador intenta devolver una respuesta lo más rápido posible (en lugar de intentar crear un ejemplo justo).

```kusto
T | sample-distinct 5 of DeviceId
```

**Sintaxis**

*T* `| sample-distinct` *NumberOfValues* `of` *columnName*

**Argumentos**
* *NumberOfValues*: el número DISTINCT Values de *T* que se va a devolver. Puede especificar cualquier expresión numérica.

**Sugerencias**

 Puede ser útil para tomar una muestra de una población colocando `sample-distinct` una instrucción Let y filtrar posteriormente mediante el `in` operador (vea el ejemplo). 

 Si desea los valores superiores en lugar de solo un ejemplo, puede usar el operador [Top-Hitters](tophittersoperator.md) 

 Si desea que muestre las filas de datos (en lugar de los valores de una columna específica), consulte el [operador de ejemplo](sampleoperator.md) .

**Ejemplos**  

Obtener 10 valores distintos de un rellenado

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents | sample-distinct 10 of EpisodeId

```

El hecho de aplicar un muestreo sobre una población y realizar cálculos adicionales conociendo el resumen no superará los límites de la consulta. 

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let sampleEpisodes = StormEvents | sample-distinct 10 of EpisodeId;
StormEvents 
| where EpisodeId in (sampleEpisodes) 
| summarize totalInjuries=sum(InjuriesDirect) by EpisodeId
```

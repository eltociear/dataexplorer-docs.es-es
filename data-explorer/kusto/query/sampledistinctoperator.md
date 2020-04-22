---
title: 'Operador de ejemplo distinto: Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de ejemplo distinto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b6d6c77aef3a7e2c6d99af792062d9f1a6215f51
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663646"
---
# <a name="sample-distinct-operator"></a>Operador sample-distinct

Devuelve una única columna que contiene hasta el número especificado de valores distintos de la columna solicitada. 

el sabor predeterminado (y actualmente solo) del operador intenta devolver una respuesta lo más rápido posible (en lugar de intentar hacer una muestra justa)

```kusto
T | sample-distinct 5 of DeviceId
```

**Sintaxis**

*T* `| sample-distinct` *NumberOfValues* `of` *ColumnName*

**Argumentos**
* *NumberOfValues*: El número de valores distintos de *T* que se va a devolver. Puede especificar cualquier expresión numérica.

**Sugerencias**

 Puede ser útil para probar `sample-distinct` una población colocando `in` una instrucción let y un filtro posterior mediante el operador (consulte el ejemplo) 

 Si desea los valores superiores en lugar de solo una muestra, puede utilizar el operador [top-hitters](tophittersoperator.md) 

 si desea muestrear filas de datos (en lugar de valores de una columna específica), consulte el operador de [ejemplo](sampleoperator.md)

**Ejemplos**  

Obtener 10 valores distintos de una población

```kusto
StormEvents | sample-distinct 10 of EpisodeId

```

El hecho de aplicar un muestreo sobre una población y realizar cálculos adicionales conociendo el resumen no superará los límites de la consulta. 

```kusto
let sampleEpisodes = StormEvents | sample-distinct 10 of EpisodeId;
StormEvents 
| where EpisodeId in (sampleEpisodes) 
| summarize totalInjuries=sum(InjuriesDirect) by EpisodeId
```
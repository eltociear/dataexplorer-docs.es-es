---
title: 'complemento de diffpatterns_text: Azure Explorador de datos'
description: En este artículo se describe diffpatterns_text complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7ef4bf5607979cc02976d00250e8754f3a0c4e69
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225180"
---
# <a name="diffpatterns_text-plugin"></a>complemento de diffpatterns_text

Compara dos conjuntos de datos de valores de cadena y busca patrones de texto que caracterizan las diferencias entre los dos conjuntos de datos.

```kusto
T | evaluate diffpatterns_text(TextColumn, BooleanCondition)
```

`diffpatterns_text`Devuelve un conjunto de patrones de texto que capturan diferentes partes de los datos de los dos conjuntos (es decir, un patrón que captura un gran porcentaje de las filas cuando la condición es `true` y un porcentaje bajo de las filas cuando la condición es `false` ). Los patrones se crean a partir de tokens consecutivos (separados por espacios en blanco), con un token de la columna de texto o un `*` que representa un carácter comodín. Cada patrón se representa mediante una fila en los resultados.

**Sintaxis**

`T | evaluate diffpatterns_text(`TextColumn, BooleanCondition [, MinTokens, threshold, MaxTokens]`)` 

**Argumentos necesarios**

* *Column_name* de TextColumn

    La columna de texto que se va a analizar debe ser de tipo cadena.
    
* BooleanCondition: *expresión booleana*

    Define cómo generar los dos subconjuntos de registros que se van a comparar con la tabla de entrada. El algoritmo divide la consulta en dos conjuntos de datos, "true" y "false" según la condición y, a continuación, analiza las diferencias de (texto) entre ellos. 

**Argumentos opcionales**

Todos los demás argumentos son opcionales, pero se deben ordenar como se indica a continuación. 

* MinTokens-0 < *int* < 200 [predeterminado: 1]

    Establece el número mínimo de tokens no comodín por patrón de resultado.

* Umbral-0,015 < *doble* < 1 [valor predeterminado: 0,05]

    Establece la diferencia de patrón mínimo (relación) entre los dos conjuntos (vea [diffpatterns](diffpatternsplugin.md)).

* MaxTokens-0 < *int* [valor predeterminado: 20]

    Establece el número máximo de tokens (desde el principio) por patrón de resultado y, al especificar un límite inferior, se reduce el tiempo de ejecución de la consulta.

**Devuelve**

El resultado de diffpatterns_text devuelve las columnas siguientes:

* Count_of_True: el número de filas que coinciden con el patrón cuando la condición es `true` .
* Count_of_False: el número de filas que coinciden con el patrón cuando la condición es `false` .
* Percent_of_True: el porcentaje de filas que coinciden con el patrón de las filas cuando la condición es `true` .
* Percent_of_False: el porcentaje de filas que coinciden con el patrón de las filas cuando la condición es `false` .
* Pattern: el patrón de texto que contiene los tokens de la cadena de texto y ' `*` ' para los caracteres comodín. 

> [!NOTE]
> Los patrones no son necesariamente distintos y es posible que no proporcionen una cobertura completa del conjunto de datos. Los patrones pueden estar superpuestos y es posible que algunas filas no coincidan con ningún patrón.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents     
| where EventNarrative != "" and monthofyear(StartTime) > 1 and monthofyear(StartTime) < 9
| where EventType == "Drought" or EventType == "Extreme Cold/Wind Chill"
| evaluate diffpatterns_text(EpisodeNarrative, EventType == "Extreme Cold/Wind Chill", 2)
```

|Count_of_True|Count_of_False|Percent_of_True|Percent_of_False|Patrón|
|---|---|---|---|---|
|11|0|6,29|0|Vientos al mover el noroeste en * Wake * una superficie de carga pesada de efecto de lago Snowfall downwind * Lake superior desde|
|9|0|5,14|0|La alta presión canadiense liquidada * * región * produjo las temperaturas más frías desde febrero * 2006. Duraciones * temperaturas de congelación|
|0|34|0|6,24|* * * * * * * * * * * * * * * * * * Oeste de Tennessee,|
|0|42|0|7,71|* * * * * * se produjo * * * * * * * * en la zona occidental de Colorado. *|
|0|45|0|8,26|* * por debajo de lo normal *|
|0|110|0|20,18|Por debajo de lo normal *|


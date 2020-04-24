---
title: diffpatterns_text complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe diffpatterns_text complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 58f059e2346dfb3f15bff295126a14a97f10f317
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516021"
---
# <a name="diffpatterns_text-plugin"></a>diffpatterns_text plugin

Compara dos conjuntos de datos de valores de cadena y busca patrones de texto que caracterizan las diferencias entre los dos conjuntos de datos.

```kusto
T | evaluate diffpatterns_text(TextColumn, BooleanCondition)
```

Devuelve `diffpatterns_text` un conjunto de patrones de texto que capturan diferentes partes de los datos de los dos conjuntos (es decir, un patrón que captura un gran porcentaje de las filas cuando la condición es `true` y un porcentaje bajo de las filas cuando la condición es `false`). Los patrones se crean a partir de tokens consecutivos (separados `*` por espacios en blanco), con un token de la columna de texto o un que representa un comodín. Cada patrón se representa mediante una fila en los resultados.

**Sintaxis**

`T | evaluate diffpatterns_text(`TextColumn, BooleanCondition [, MinTokens, Threshold , MaxTokens]`)` 

**Argumentos requeridos**

* TextColumn - *column_name*

    La columna de texto que se va a analizar debe ser de tipo string.
    
* BooleanCondition - *Expresión booleana*

    Define cómo generar los dos subconjuntos de registros para comparar con la tabla de entrada. El algoritmo divide la consulta en dos conjuntos de datos, "True" y "False" según la condición y, a continuación, analiza las diferencias (texto) entre ellos. 

**Argumentos opcionales**

Todos los demás argumentos son opcionales, pero se deben ordenar como se indica a continuación. 

* MinTokens - 0 < *int* < 200 [predeterminado: 1]

    Establece el número mínimo de tokens no comodín por patrón de resultados.

* Umbral - 0.015 < *doble* < 1 [predeterminado: 0.05]

    Establece la diferencia mínima de patrón (relación) entre los dos conjuntos (consulte [diffpatterns](diffpatternsplugin.md)).

* MaxTokens - 0 < *int* [predeterminado: 20]

    Establece el número máximo de tokens (desde el principio) por patrón de resultados, especificar un límite inferior disminuye el tiempo de ejecución de la consulta.

**Devuelve**

El resultado de diffpatterns_text devuelve las siguientes columnas:

* Count_of_True: el número de filas que `true`coinciden con el patrón cuando la condición es .
* Count_of_False: el número de filas que `false`coinciden con el patrón cuando la condición es .
* Percent_of_True: el porcentaje de filas que coinciden con `true`el patrón de las filas cuando la condición es .
* Percent_of_False: el porcentaje de filas que coinciden con `false`el patrón de las filas cuando la condición es .
* Patrón: el patrón de texto`*`que contiene tokens de la cadena de texto y ' ' para comodines. 

> [!NOTE]
> Los patrones no son necesariamente distintos y pueden no proporcionar una cobertura completa del conjunto de datos. Los patrones pueden estar superpuestos y algunas filas pueden no coincidir con ningún patrón.

**Ejemplo**

```kusto
StormEvents     
| where EventNarrative != "" and monthofyear(StartTime) > 1 and monthofyear(StartTime) < 9
| where EventType == "Drought" or EventType == "Extreme Cold/Wind Chill"
| evaluate diffpatterns_text(EpisodeNarrative, EventType == "Extreme Cold/Wind Chill", 2)
```
|Count_of_True|Count_of_False|Percent_of_True|Percent_of_False|Modelo|
|---|---|---|---|---|
|11|0|6.29|0|Vientos que se desplazan hacia el noroeste en * despertar * un vaguada de superficie trajo pesado efecto lago nevada sin viento * Lago Superior de|
|9|0|5.14|0|La alta presión canadiense se asentó * * región * produjo las temperaturas más frías desde febrero * 2006. Duraciones * temperaturas de congelación|
|0|34|0|6.24|* * * * * * * * * * * * * * * * * * West Tennessee,|
|0|42|0|7.71|* * * * * causado * * * * * * * * en todo el oeste de Colorado. *|
|0|45|0|8.26|* * por debajo de lo normal *|
|0|110|0|20.18|Por debajo de lo normal *|


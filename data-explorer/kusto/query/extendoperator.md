---
title: 'operador de extensión: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador extend en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4b9b7bdb9488b0e9d1b72b3e0ab4782020c9b841
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515596"
---
# <a name="extend-operator"></a>Operador extend

Cree columnas calculadas y anótelas al conjunto de resultados.

```kusto
T | extend duration = endTime - startTime
```

**Sintaxis**

*T* `| extend` [*ColumnName* | `(``,` *ColumnName*[ ...] ] Expresión`,` [ ...] *Expression* `)` `=`

**Argumentos**

* *T*: El conjunto de resultados tabulares de entrada.
* *ColumnName:* Opcional. El nombre de la columna que se va a agregar o actualizar. Si se omite, se generará el nombre. Si *Expresión* devuelve más de una columna, se puede especificar una lista de nombres de columna entre paréntesis. En este caso, las columnas de salida de *Expression*'s recibirán los nombres especificados, quitando el resto de las columnas de salida, si las hay. Si no se especifica una lista de los nombres de columna, todas las columnas de salida de *Expression*con nombres generados se agregarán a la salida.
* *Expresión:* Un cálculo sobre las columnas de la entrada.

**Devuelve**

Una copia del conjunto de resultados tabulares de entrada, de modo que:
1. Los nombres `extend` de columna indicados por los que ya existen en la entrada se quitan y se anexan como sus nuevos valores calculados.
2. Los nombres `extend` de columna indicados por los que no existen en la entrada se anexan como sus nuevos valores calculados.

**Sugerencias**

* El `extend` operador agrega una nueva columna al conjunto de resultados de entrada, que **no** tiene un índice. En la mayoría de los casos, si la nueva columna se establece para que sea exactamente la misma que una columna de tabla existente que tiene un índice, Kusto puede usar automáticamente el índice existente. Sin embargo, en algunos escenarios complejos esta propagación no se realiza. En tales casos, si el objetivo es [ `project-rename` ](projectrenameoperator.md) cambiar el nombre de una columna, utilice el operador en su lugar.

**Ejemplo**

```kusto
Logs
| extend
    Duration = CreatedOn - CompletedOn
    , Age = now() - CreatedOn
    , IsSevere = Level == "Critical" or Level == "Error"
```

Puede utilizar la función [series_stats](series-statsfunction.md) para devolver varias columnas.
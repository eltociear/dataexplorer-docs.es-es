---
title: 'Operador del proyecto: Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador Project en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: de13c240180d00b82736a0dd35cb83c08639682f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510938"
---
# <a name="project-operator"></a>Operador del proyecto

Seleccione las columnas que se incluirán, cambie el nombre o quite e inserte nuevas columnas calculadas. 

El orden de las columnas en el resultado se especifica con el orden de los argumentos. Solo las columnas especificadas en los argumentos se incluyen en el resultado. Cualquier otra columna de la entrada se descarta.  (Consulte también `extend`).

```kusto
T | project cost=price*quantity, price
```

**Sintaxis**

*T* `| project` *ColumnName* [`=` `,` *Expresión*] [ ...]
  
or
  
*T* `| project` [*ColumnName* | `(``,`*ColumnName*[`,` ]`)` `=`] *Expresión* [ ...]

**Argumentos**

* *T*: La tabla de entrada.
* *ColumnName:* Nombre opcional de una columna para que aparezca en la salida. Si no hay *ninguna expresión*, *ColumnName* es obligatorio y debe aparecer una columna con ese nombre en la entrada. Si se omite, el nombre se generará automáticamente. Si *Expresión* devuelve más de una columna, se puede especificar una lista de nombres de columna entre paréntesis. En este caso, las columnas de salida de *Expression*'s recibirán los nombres especificados, quitando el resto de las columnas de salida, si las hay. Si no se especifica la lista de nombres de columna, todas las columnas de salida de *Expression*con nombres generados se agregarán a la salida.
* *Expression:* expresión escalar opcional que hace referencia a las columnas de entrada. Si no se omite *ColumnName,* *Expression* es obligatorio.

    Es posible devolver una nueva columna calculada con el mismo nombre que una columna existente en la entrada.

**Devuelve**

Una tabla que tiene las columnas con nombres de argumentos y tantas filas como la tabla de entrada.

**Ejemplo**

El ejemplo siguiente muestra variantes de manipulaciones que se pueden hacer con el operador `project` . La tabla de entrada `T` tiene tres columnas de tipo `int`: `A`, `B` y `C`. 

```kusto
T
| project
    X=C,                       // Rename column C to X
    A=2*B,                     // Calculate a new column A from the old B
    C=strcat("-",tostring(C)), // Calculate a new column C from the old C
    B=2*B                      // Calculate a new column B from the old B
```

[series_stats](series-statsfunction.md) es un ejemplo de una función que devuelve varias columnas.
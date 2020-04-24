---
title: 'Operador de búsqueda: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de búsqueda en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: 1325a1b839b62baacade4cf7c64cd278aeeabaf5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513063"
---
# <a name="lookup-operator"></a>Operador lookup

El `lookup` operador extiende las columnas de una tabla de hechos con valores buscados en una tabla de dimensiones.

```kusto
FactTable | lookup kind=leftouter (DimensionTable) on CommonColumn, $left.Col1 == $right.Col2
```

Aquí, el resultado es una `FactTable` tabla`$left`que extiende `DimensionTable` el `$right`( ) con datos de`CommonColumn`(referenciado por ) realizando una búsqueda de cada par ( ,`Col`) de la tabla anterior con cada par (`CommonColumn1`,`Col2`) en la última tabla. Para conocer las diferencias entre las tablas de hechos y dimensiones, consulte Tablas de [hechos y dimensiones.](../concepts/fact-and-dimension-tables.md) 

El `lookup` operador realiza una operación similar al [operador de combinación](joinoperator.md) con las siguientes diferencias:

* El resultado no repite `$right` columnas de la tabla que son la base de la operación de combinación.
* Solo se admiten dos `leftouter` `inner`tipos `leftouter` de búsqueda y , con el valor predeterminado.
* En términos de rendimiento, el sistema `$left` asume de forma predeterminada que `$right` la tabla es la tabla (hechos) más grande y la tabla es la tabla (dimensiones) más pequeña. Esto es exactamente opuesto a `join` la suposición utilizada por el operador.
* El `lookup` operador difunde `$right` automáticamente `$left` la tabla a la tabla `hint.broadcast` (esencialmente, se comporta como si se especificara). Tenga en cuenta que `$right` esto limita el tamaño de la tabla.

**Sintaxis**

*LeftTable* `|` `lookup` `kind` `=` [`leftouter`|`inner`( `(` )] `)` `on` *Atributos* *rightTable*

**Argumentos**

* *LeftTable*: La tabla o expresión tabular que es la base para la búsqueda.
  Denotado `$left`como .

* *RightTable*: la tabla o expresión tabular que se utiliza para "rellenar" nuevas columnas en la tabla de hechos. Denotado `$right`como .

* *Atributos*: Una lista delimitada por comas de una o más reglas que describen cómo las filas de *LeftTable* coinciden con las filas de *RightTable*. Se evalúan varias `and` reglas mediante el operador lógico.
  Una regla puede ser una de las:

  |Tipo de regla        |Sintaxis                                          |Predicate                                                      |
  |-----------------|------------------------------------------------|---------------------------------------------------------------|
  |Igualdad por nombre |*ColumnName*                                    |`where`*LeftTable*. *ColumnName* `==` *RightTable*. *ColumnName*|
  |Igualdad por valor|`$left.`*LeftColumn* `==` `$right.` *RightColumn*|`where``$left.` `$right.` *LeftColumn* `==` *RightColumn        |

  > [!Note] 
  > En caso de "igualdad por valor", los nombres de columna *deben* calificarse con la tabla de propietario aplicable denotada por `$left` y `$right` notaciones.

* `kind`: una instrucción opcional sobre cómo tratar filas en *LeftTable* que no tienen ninguna coincidencia en *RightTable*. De forma `leftouter` predeterminada, se utiliza, lo que significa que todas esas filas aparecerán en la salida con valores nulos utilizados para los valores que faltan de las columnas *RightTable* agregadas por el operador. Si `inner` se utiliza, estas filas se omiten de la salida. (Otros tipos de combinación `looku`no son compatibles con el operador p.)
  
**Devuelve**

Una tabla con:

* Una columna por cada columna en cada una de las dos tablas, incluidas las claves coincidentes.
  Las columnas del lado derecho se cambiarán automáticamente si hay conflictos de nombre.
* Una fila por cada correspondencia entre las tablas de entrada. Una coincidencia es una fila seleccionada de una tabla que tiene el mismo valor para todos los campos `on` que una fila en la otra tabla. 
* Los atributos (claves de búsqueda) aparecerán solo una vez en la tabla de salida.

 * `kind`Indeterminado`kind=leftouter`

     Además de las coincidencias internas, hay una fila por cada fila de la izquierda (o derecha), incluso si no tiene ninguna coincidencia. En este caso, las celdas de salida sin coincidencias contienen valores NULL.

 * `kind=inner`

     Hay una fila en la salida por cada combinación de filas coincidentes de la izquierda y la derecha.

**Ejemplos**

```kusto
let FactTable=datatable(Row:string,Personal:string,Family:string) [
  "1", "Bill",   "Gates",
  "2", "Bill",   "Clinton",
  "3", "Bill",   "Clinton",
  "4", "Steve",  "Ballmer",
  "5", "Tim",    "Cook"
];
let DimTable=datatable(Personal:string,Family:string,Alias:string) [
  "Bill",  "Gates",   "billg",
  "Bill",  "Clinton", "billc",
  "Steve", "Ballmer", "steveb",
  "Tim",   "Cook",    "timc"
];
FactTable
| lookup kind=leftouter DimTable on Personal, Family
```

Row     | Personal  | Familia   | Alias
--------|-----------|----------|--------
1       | Bill      | Puertas    | billg
2       | Bill      | Clinton  | billc
3       | Bill      | Clinton  | billc
4       | Steve     | Ballmer  | steveb
5       | Tim       | Cocinar     | timc
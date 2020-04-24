---
title: 'operador de reducción: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe el operador de reducción en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 33d4ac202b61fdaa1b92291407cdd2783d947c6e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510513"
---
# <a name="reduce-operator"></a>Operador reduce

Agrupa un conjunto de cadenas en función de la similitud de valores.

```kusto
T | reduce by LogMessage with threshold=0.1
```

Para cada uno de estos grupos, genera un **patrón** que describe`*`mejor el grupo (posiblemente utilizando el carácter asterix ( ) para representar comodines), un **recuento** del número de valores del grupo y un **representante** del grupo (uno de los valores originales del grupo).

**Sintaxis**

*T* `|` `,` `characters` `=` `threshold` `=` *Threshold*`with` *ReduceKind* `by` *Expr* *Characters*[`kind` `=` ReduceKind ] Expr [ [ Umbral ] [ Caracteres ] ] `reduce`

**Argumentos**

* *Expr*: una expresión que `string` se evalúa como un valor.
* *Umbral*: `real` Un literal en el rango (0..1). El valor predeterminado es 0.1. Para las entradas grandes, el valor threshold debe ser pequeño. 
* *Caracteres* `string` : Literal que contiene una lista de caracteres para agregar a la lista de caracteres que no rompen un término. (Por ejemplo, si `aaa=bbbb` `aaa:bbb` desea y a cada uno ser `=` `:`un `":="` término completo, en lugar de interrumpir y , utilizar como la cadena literal.)
* *ReduceKind*: Especifica el sabor de reducción. El único valor válido para `source`el momento es .

**Devuelve**

Este operador devuelve una tabla`Pattern` `Count`con `Representative`tres columnas ( , , y ), y tantas filas como grupos haya grupos. `Pattern`es el valor de patrón para el grupo, con `*` que `Count` se utiliza como comodín (que representa cadenas de `Representative` inserción arbitrarias), cuenta cuántas filas de la entrada al operador están representadas por este patrón y es un valor de la entrada que cae en este grupo.

Si `[kind=source]` se especifica, el operador `Pattern` anexará la columna a la estructura de tabla existente.
Tenga en cuenta que la sintaxis de un esquema de este sabor podría estar sujeta a cambios futuros.

Por ejemplo, el resultado de `reduce by city` podría incluir: 

|Modelo     |Count |Representative|
|------------|------|--------------|
| San *      | 5182 |San Bernard   |
| Saint *    | 2846 |Santa Lucy    |
| Moscow     | 3726 |Moscow        |
| \* -on- \* | 2730 |Uno -on- Uno  |
| Paris      | 2716 |Paris         |

Otro ejemplo con tokenización personalizada:

```kusto
range x from 1 to 1000 step 1
| project MyText = strcat("MachineLearningX", tostring(toint(rand(10))))
| reduce by MyText  with threshold=0.001 , characters = "X" 
```

|Modelo         |Count|Representative   |
|----------------|-----|-----------------|
|MachineLearning*|1000 |MachineLearningX4|

**Ejemplos**

En el ejemplo siguiente se `reduce` muestra cómo se puede aplicar el operador a una entrada "sanitizada", en la que los GUID de la columna que se va a reducir se reemplazan antes de reducir

```kusto
// Start with a few records from the Trace table.
Trace | take 10000
// We will reduce the Text column which includes random GUIDs.
// As random GUIDs interfere with the reduce operation, replace them all
// by the string "GUID".
| extend Text=replace(@"[[:xdigit:]]{8}-[[:xdigit:]]{4}-[[:xdigit:]]{4}-[[:xdigit:]]{4}-[[:xdigit:]]{12}", @"GUID", Text)
// Now perform the reduce. In case there are other "quasi-random" identifiers with embedded '-'
// or '_' characters in them, treat these as non-term-breakers.
| reduce by Text with characters="-_"
```

**Vea también**

[autocluster](./autoclusterplugin.md)

**Notas**

La implementación del `reduce` operador se basa en gran medida en el papel A Data [Clustering Algorithm for Mining Patterns From Event Logs](https://ristov.github.io/publications/slct-ipom03-web.pdf), de Risto Vaarandi.
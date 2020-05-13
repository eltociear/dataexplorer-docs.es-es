---
title: 'reducción del operador: Azure Explorador de datos'
description: En este artículo se describe el operador de reducción en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7d157244a167e1264b454cd8cd3c103e297c3263
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373052"
---
# <a name="reduce-operator"></a>Operador reduce

Agrupa un conjunto de cadenas en función de la similitud de los valores.

```kusto
T | reduce by LogMessage with threshold=0.1
```

En cada uno de estos grupos, se genera un **patrón** que describe mejor el grupo (posiblemente mediante el `*` carácter asterisco () para representar caracteres comodín), un **recuento** del número de valores del grupo y un **representante** del grupo (uno de los valores originales del grupo).

**Sintaxis**

*T* `|` `reduce` [ `kind` `=` *ReduceKind*] `by` *expr* [ `with` [ `threshold` `=` *umbral*] [ `,` `characters` `=` *caracteres*]]

**Argumentos**

* *Expr*: expresión que se evalúa como un `string` valor.
* *Threshold*: un `real` literal en el intervalo (0.. 1). El valor predeterminado es 0,1. Para las entradas grandes, el valor threshold debe ser pequeño. 
* *Characters*: `string` literal que contiene una lista de caracteres que se van a agregar a la lista de caracteres que no interrumpen un término. (Por ejemplo, si desea que `aaa=bbbb` `aaa:bbb` cada uno sea un término completo, en lugar de interrumpir en `=` y `:` , use `":="` como literal de cadena).
* *ReduceKind*: especifica el tipo de reducción. El único valor válido para el momento es `source` .

**Devuelve**

Este operador devuelve una tabla con tres columnas ( `Pattern` , `Count` y `Representative` ) y tantas filas como grupos haya. `Pattern`es el valor de patrón del grupo, `*` que se usa como carácter comodín (que representa cadenas de inserción arbitrarias), `Count` cuenta el número de filas de la entrada para el operador que este patrón representa y `Representative` es un valor de la entrada que entra en este grupo.

Si `[kind=source]` se especifica, el operador anexará la `Pattern` columna a la estructura de tabla existente.
Tenga en cuenta que la sintaxis de un esquema de este tipo puede estar sujeta a cambios futuros.

Por ejemplo, el resultado de `reduce by city` podría incluir: 

|Patrón     |Count |Representative|
|------------|------|--------------|
| San *      | 5182 |San Bernard   |
| Saint *    | 2846 |San Lucy    |
| Moscow     | 3726 |Moscow        |
| \* -on- \* | 2730 |Uno a uno  |
| Paris      | 2716 |Paris         |

Otro ejemplo con la tokenización personalizada:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 1000 step 1
| project MyText = strcat("MachineLearningX", tostring(toint(rand(10))))
| reduce by MyText  with threshold=0.001 , characters = "X" 
```

|Patrón         |Count|Representative   |
|----------------|-----|-----------------|
|MachineLearning|1000 |MachineLearningX4|

**Ejemplos**

En el ejemplo siguiente se muestra cómo se puede aplicar el `reduce` operador a una entrada "SANEADO", en la que los GUID de la columna que se está reduciendo se reemplazan antes de reducir

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

La implementación del `reduce` operador se basa en gran medida en el documento [un algoritmo de agrupación en clústeres de datos para patrones de minería de datos de registros de eventos](https://ristov.github.io/publications/slct-ipom03-web.pdf), por Risto Vaarandi.

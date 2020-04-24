---
title: donde operator (tiene, contiene, startswith, endswith, coincide con regex) - Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe dónde operador (tiene, contiene, comienza, termina con regex) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a0e1486691740600fb5de4316982e8d43b13ce3a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504308"
---
# <a name="where-operator-has-contains-startswith-endswith-matches-regex"></a>donde operador (tiene, contiene, comienza, termina, coincide con regex)

Filtra una tabla para el subconjunto de filas que cumplen un predicado.

```kusto
T | where fruit=="apple"
```

**Alias** `filter`

**Sintaxis**

*T* `| where` *Predicate*

**Argumentos**

* *T*: La entrada tabular cuyos registros se van a filtrar.
* *Predicado*: Una `boolean` [expresión](./scalar-data-types/bool.md) sobre las columnas de *T*. Se evalúa para cada fila en *T*.

**Devuelve**

Las filas de *T* en las que *Predicate* es `true`.

**Notas** Valores nulos: todas las funciones de filtrado devuelven false cuando se comparan con valores nulos. Puede utilizar funciones especiales que reconocen los valores nulos para escribir consultas que tienen en cuenta valores nulos: [isnull()](./isnullfunction.md), [isnotnull()](./isnotnullfunction.md), [isempty()](./isemptyfunction.md), [isnotempty()](./isnotemptyfunction.md). 

**Sugerencias**

Para obtener el rendimiento más rápido:

* **Use comparaciones simples** entre los nombres de columna y las constantes. ('Constante' significa constante sobre `now()` la `ago()` tabla - así y están bien, al igual que los valores escalares asignados mediante una [ `let` instrucción](./letstatement.md).)

    Por ejemplo, se prefiere `where Timestamp >= ago(1d)` a `where floor(Timestamp, 1d) == ago(1d)`.

* **Primero los términos más simples**: si tiene varias cláusulas unidas con `and`, coloque primero las cláusulas que impliquen una sola columna. Así pues, es mejor usar `Timestamp > ago(1d) and OpId == EventId` que al revés.

Para obtener más información, consulte el resumen de [los operadores String disponibles](./datatypes-string-operators.md) y el resumen de [los operadores numéricos disponibles.](./numoperators.md)

**Ejemplo**

```kusto
Traces
| where Timestamp > ago(1h)
    and Source == "MyCluster"
    and ActivityId == SubActivityId 
```

Los registros que no tienen más de 1 hora y proceden del origen denominado "MyCluster" y tienen dos columnas del mismo valor. 

Observe que colocamos la comparación entre dos columnas al final, ya que no puede utilizar el índice y exige un examen.

**Ejemplo**

```kusto
Traces | where * has "Kusto"
```

Todas las filas en las que aparece la palabra "Kusto" en cualquier columna.
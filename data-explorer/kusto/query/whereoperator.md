---
title: 'operador where en el lenguaje de consulta de Kusto: Azure Explorador de datos'
description: En este artículo se describe el operador where en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7dc9d7166a1f286e14c81f269f32f894cbe9ff9d
ms.sourcegitcommit: da7c699bb62e1c4564f867d4131d26286c5223a8
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2020
ms.locfileid: "83404169"
---
# <a name="where-operator"></a>Operador where

Filtra una tabla para el subconjunto de filas que cumplen un predicado.

```kusto
T | where fruit=="apple"
```

**Alias** de`filter`

**Sintaxis**

*T* ( `| where` *predicado* )

**Argumentos**

* *T*: la entrada tabular cuyos registros se van a filtrar.
* *Predicate*: una `boolean` [expresión](./scalar-data-types/bool.md) sobre las columnas de *T*. Se evalúa para cada fila en *T*.

**Devuelve**

Las filas de *T* en las que *Predicate* es `true`.

**Notas** de Valores NULL: todas las funciones de filtrado devuelven false cuando se comparan con valores NULL. Puede usar funciones especiales que reconocen valores NULL para escribir consultas que toman valores NULL en la cuenta: [IsNull ()](./isnullfunction.md), [isnotnull ()](./isnotnullfunction.md), [isEmpty ()](./isemptyfunction.md), [isnotempty ()](./isnotemptyfunction.md). 

**Sugerencias**

Para obtener el rendimiento más rápido:

* **Use comparaciones simples** entre los nombres de columna y las constantes. (' Constant ' significa constante sobre la tabla, por lo que `now()` y `ago()` son correctos y, por tanto, son valores escalares asignados mediante una [ `let` instrucción](./letstatement.md)).

    Por ejemplo, se prefiere `where Timestamp >= ago(1d)` a `where floor(Timestamp, 1d) == ago(1d)`.

* **Primero los términos más simples**: si tiene varias cláusulas unidas con `and`, coloque primero las cláusulas que impliquen una sola columna. Así pues, es mejor usar `Timestamp > ago(1d) and OpId == EventId` que al revés.

Para obtener más información, vea el Resumen de los [operadores de cadena disponibles](./datatypes-string-operators.md) y el Resumen de los [operadores numéricos disponibles](./numoperators.md).

**Ejemplo**

```kusto
Traces
| where Timestamp > ago(1h)
    and Source == "MyCluster"
    and ActivityId == SubActivityId 
```

En este ejemplo se recuperan registros que no tienen más de 1 hora, provienen de un origen llamado `MyCluster` y tienen dos columnas con el mismo valor. 

Observe que colocamos la comparación entre dos columnas en último lugar, ya que no puede utilizar el índice y fuerza un examen.

**Ejemplo**

```kusto
Traces | where * has "Kusto"
```

Todas las filas en las que aparece la palabra "Kusto" en cualquier columna.

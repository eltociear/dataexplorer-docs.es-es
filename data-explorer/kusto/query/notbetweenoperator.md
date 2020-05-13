---
title: 'operador notbetween: Azure Explorador de datos'
description: En este artículo se describe el operador notbetween en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: e951e56589824acd6dd74160c9ab61778fbce45e
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271560"
---
# <a name="between-operator"></a>! Between (operador)

Coincide con la entrada que está fuera del intervalo inclusivo.

```kusto
Table1 | where Num1 !between (1 .. 10)
Table1 | where Time !between (datetime(2017-01-01) .. datetime(2017-01-01))
```

`!between`puede operar en cualquier expresión numérica, DateTime o TimeSpan.
 
**Sintaxis**

*T* `|` `where` *expr* `!between` `(` *leftRange* ` .. ` *rightRange*`)`   
 
Si *expr* Expression es DateTime, se proporciona otra sintaxis de azúcar sintáctico:

*T* `|` `where` *expr* `!between` `(` *leftRangeDateTime* ` .. ` *rightRangeTimespan*`)`   

**Argumentos**

* *T* : la entrada tabular cuyos registros deben coincidir.
* *expr* : expresión que se va a filtrar.
* *leftRange* : expresión del intervalo izquierdo (inclusivo).
* *rightRange* : expresión del intervalo de RIHGT (inclusivo).

**Devuelve**

Filas de *T* para las que el predicado de (*expr*  <  *leftRange* o *expr*  >  *rightRange*) se evalúa como `true` .

**Ejemplos**  

**Filtrar valores numéricos mediante el operador '! between '**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 10 step 1
| where x !between (5 .. 9)
```

|x|
|---|
|1|
|2|
|3|
|4|
|10|

**Filtrar DateTime mediante el operador ' between '**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| where StartTime !between (datetime(2007-07-27) .. datetime(2007-07-30))
| count 
```

|Count|
|---|
|58590|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| where StartTime !between (datetime(2007-07-27) .. 3d)
| count 
```

|Count|
|---|
|58590|

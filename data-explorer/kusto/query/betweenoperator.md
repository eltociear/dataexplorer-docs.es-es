---
title: 'Between (operador): Azure Explorador de datos'
description: En este artículo se describe entre el operador en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: ef64818c9c5e345ffb60999c97273670026be022
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227628"
---
# <a name="between-operator"></a>Operador between

coincide con la entrada que está dentro del intervalo inclusivo.

```kusto
Table1 | where Num1 between (1 .. 10)
Table1 | where Time between (datetime(2017-01-01) .. datetime(2017-01-01))
```

`between`puede operar en cualquier expresión numérica, DateTime o TimeSpan.
 
**Sintaxis**

*T* `|` `where` *expr* `between` `(` *leftRange* ` .. ` *rightRange*`)`   
 
Si *expr* Expression es DateTime, se proporciona otra sintaxis de azúcar sintáctico:

*T* `|` `where` *expr* `between` `(` *leftRangeDateTime* ` .. ` *rightRangeTimespan*`)`   

**Argumentos**

* *T* : la entrada tabular cuyos registros deben coincidir.
* *expr* : expresión que se va a filtrar.
* *leftRange* : expresión del intervalo izquierdo (inclusivo).
* *rightRange* : expresión del intervalo derecho (inclusivo).

**Devuelve**

Filas de *T* para las que el predicado de (*expr*  >=  *leftRange* y *expr*  <=  *rightRange*) se evalúa como `true` .

**Ejemplos**  

**Filtrar valores numéricos mediante el operador ' between '**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 100 step 1
| where x between (50 .. 55)
```

|x|
|---|
|50|
|51|
|52|
|53|
|54|
|55|

**Filtrar DateTime mediante el operador ' between '**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. datetime(2007-07-30))
| count 
```

|Count|
|---|
|476|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. 3d)
| count 
```

|Count|
|---|
|476|

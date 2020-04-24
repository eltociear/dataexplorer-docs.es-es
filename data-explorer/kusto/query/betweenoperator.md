---
title: entre el operador- Explorador de datos de Azure - Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe entre el operador en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 336d24edbcd7574f0c4b6375b4b09014b38d10ec
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517857"
---
# <a name="between-operator"></a>Operador between

coincide con la entrada que está dentro del intervalo inclusivo.

```kusto
Table1 | where Num1 between (1 .. 10)
Table1 | where Time between (datetime(2017-01-01) .. datetime(2017-01-01))
```

`between`puede funcionar en cualquier expresión numérica, datetime o timespan.
 
**Sintaxis**

*T* `|` T `where` *expr* `between` *leftRange*` .. `*rightRange* leftRange rightRange `(``)`   
 
Si la expresión *expr* es datetime - se proporciona otra sintaxis de azúcar sintáctica:

*T* `|` T `where` *expr* `between` *leftRangeDateTime*` .. `*rightRangeTimespan* leftRangeDateTime rightRangeTimespan `(``)`   

**Argumentos**

* *T* - La entrada tabular cuyos registros deben coincidir.
* *expr* - la expresión para filtrar.
* *leftRange* - expresión del rango izquierdo (incluido).
* *rightRange* - expresión del rango correcto (incluido).

**Devuelve**

Filas en *T* para las que el predicado de ( `true`*expr* >= *leftRange* y *expr* <= *rightRange*) se evalúa como .

**Ejemplos**  

**Filtrar valores numéricos mediante el operador 'entre'**  

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

**Filtrar datetime usando el operador 'between'**  


```kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. datetime(2007-07-30))
| count 
```

|Count|
|---|
|476|


```kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. 3d)
| count 
```

|Count|
|---|
|476|
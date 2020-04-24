---
title: make_string() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_string() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d5af7cab9106088064048c1077ec3f9b1950ec08
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512638"
---
# <a name="make_string"></a>make_string()

Devuelve la cadena generada por los caracteres Unicode.
    
**Sintaxis**

`make_string (`*Arg1* [, *ArgN*]...`)`

**Argumentos**

* *Arg1* ... *ArgN* : expresiones que son enteros (int o long) o un valor dinámico que contiene una matriz de números enteros.

* Esta función recibe hasta 64 argumentos. 

**Devuelve**

Devuelve la cadena formada por los caracteres Unicode cuyo valor de punto de código proporciona los argumentos de esta función. La entrada debe constar de puntos de código Unicode válidos.
Si cualquier argumento no se asigna a un carácter Unicode, la función devuelve null.

**Ejemplos**

```kusto
print str = make_string(75, 117, 115, 116, 111)
```

|str|
|---|
|Kusto|
    
```kusto
print str = make_string(dynamic([75, 117, 115, 116, 111]))
```

|str|
|---|
|Kusto|

```kusto
print str = make_string(dynamic([75, 117, 115]), 116, 111)
```

|str|
|---|
|Kusto|

```kusto
print str = make_string(75, 10, 117, 10, 115, 10, 116, 10, 111)
```

|str|
|---|
|K<br>u<br>s<br>t<br>o|
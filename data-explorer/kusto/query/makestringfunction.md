---
title: 'make_string (): Explorador de datos de Azure'
description: En este artículo se describe make_string () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 035be5910d173c75e585baa0b093dc6276bd4d63
ms.sourcegitcommit: 3848b8db4c3a16bda91c4a5b7b8b2e1088458a3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84818585"
---
# <a name="make_string"></a>make_string()

Devuelve la cadena generada por los caracteres Unicode.
    
**Sintaxis**

`make_string (`*Arg1*[, *ArgN*]...`)`

**Argumentos**

* *Arg1* ... *ArgN*: expresiones que son enteros (int o Long) o un valor dinámico que contiene una matriz de números enteros.

* Esta función recibe hasta 64 argumentos.

**Devuelve**

Devuelve la cadena formada por los caracteres Unicode cuyo valor de punto de código proporcionan los argumentos para esta función. La entrada debe estar formada por puntos de código Unicode válidos.
Si alguno de los argumentos no está asignado a un carácter Unicode, la función devuelve `null` .

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
|K<br>u<br>s<br>m<br>o|

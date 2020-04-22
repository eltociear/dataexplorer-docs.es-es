---
title: parse_csv() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe parse_csv() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6645fd0dcc7062a4afb60fb34ade1c519af27294
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663669"
---
# <a name="parse_csv"></a>parse_csv()

Divide una cadena determinada que representa un único registro de valores separados por comas y devuelve una matriz de cadenas con estos valores.

```kusto
parse_csv("aaa,bbb,ccc") == ["aaa","bbb","ccc"]
```

**Sintaxis**

`parse_csv(`*Fuente*`)`

**Argumentos**

* *source*: La cadena de origen que representa un único registro de valores separados por comas.

**Devuelve**

Matriz de cadenas que contiene los valores divididos.

**Notas**

Las sellos de línea, comas y comillas incrustadas pueden escaparse con comillas dobles ('"'). Esta función no admite varios registros por fila (solo se toma el primer registro).

**Ejemplos**

```kusto
print result=parse_csv('aa,"b,b,b",cc,"Escaping quotes: ""Title""","line1\nline2"')
```

|resultado|
|---|
|[<br>  "aa",<br>  "b,b,b",<br>  "cc",<br>  "Escaping \"quotes:\"Title",<br>  "línea1-nline2"<br>]|

Carga útil CSV con varios registros:

```kusto
print result_multi_record=parse_csv('record1,a,b,c\nrecord2,x,y,z')
```

|result_multi_record|
|---|
|[<br>  "record1",<br>  "a",<br>  "b",<br>  "c"<br>]|
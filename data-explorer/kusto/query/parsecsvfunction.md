---
title: 'parse_csv (): Explorador de datos de Azure'
description: En este artículo se describe parse_csv () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f01faae3d9339aa23e7e2bb2b1fdae7a652db360
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271220"
---
# <a name="parse_csv"></a>parse_csv()

Divide una cadena determinada que representa un único registro de valores separados por comas y devuelve una matriz de cadenas con estos valores.

```kusto
parse_csv("aaa,bbb,ccc") == ["aaa","bbb","ccc"]
```

**Sintaxis**

`parse_csv(`*fuentes*`)`

**Argumentos**

* *source*: la cadena de origen que representa un único registro de valores separados por comas.

**Devuelve**

Matriz de cadenas que contiene los valores de división.

**Notas**

Los saltos de línea, las comas y las comillas se pueden usar con comillas dobles (' ' '). Esta función no admite varios registros por fila (solo se toma el primer registro).

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print result=parse_csv('aa,"b,b,b",cc,"Escaping quotes: ""Title""","line1\nline2"')
```

|resultado|
|---|
|[<br>  "AA",<br>  "b, b, b",<br>  "CC",<br>  "Comillas de escape: \" título \" ",<br>  "line1\nline2"<br>]|

Carga de CSV con varios registros:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print result_multi_record=parse_csv('record1,a,b,c\nrecord2,x,y,z')
```

|result_multi_record|
|---|
|[<br>  "record1",<br>  "a",<br>  "b",<br>  "c"<br>]|

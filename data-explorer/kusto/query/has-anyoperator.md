---
title: has_any operador- Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe has_any operador en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 42a181f7c75ef36119f7f2915fd5327d4a656eb8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514321"
---
# <a name="has_any-operator"></a>Operador has_any

`has_any`filtros de operador en función del conjunto de valores proporcionado.

```kusto
Table1 | where col has_any ('value1', 'value2')
```

**Sintaxis**

*T* `|` T `where` *col* `has_any` col `(` *lista de expresiones escalares*`)`   
*T* `|` T `where` *col* `has_any` col `(` *expresión tabular*`)`   
 
**Argumentos**

* *T* - Entrada tabular cuyos registros se van a filtrar.
* *col* - Columna para filtrar.
* *lista de expresiones:* lista separada por comas de expresiones tabulares, escalares o literales  
* *Expresión tabular:* expresión tabular que tiene un conjunto de valores (si la expresión tiene varias columnas, se utiliza la primera columna)

**Devuelve**

Filas en *T* para las que el predicado es`true`

**Notas**

* La lista de expresiones puede producir hasta `10,000` valores.    
* Para las expresiones tabulares, se selecciona la primera columna del conjunto de resultados.   

**Ejemplos:**  

**Un uso `has_any` sencillo del operador:**  

```kusto
StormEvents 
| where State has_any ("CAROLINA", "DAKOTA", "NEW") 
| summarize count() by State
```

|State|count_|
|---|---|
|NUEVA YORK|1750|
|CAROLINA DEL NORTE|1721|
|DAKOTA DEL SUR|1567|
|NUEVA JERSEY|1044|
|CAROLINA DEL SUR|915|
|DAKOTA DEL NORTE|905|
|NUEVO MÉXICO|527|
|NEW HAMPSHIRE|394|


**Uso de la matriz dinámica:**

```kusto
let states = dynamic(['south', 'north']);
StormEvents 
| where State has_any (states)
| summarize count() by State
```

|State|count_|
|---|---|
|CAROLINA DEL NORTE|1721|
|DAKOTA DEL SUR|1567|
|CAROLINA DEL SUR|915|
|DAKOTA DEL NORTE|905|
|ATLANTIC SOUTH|193|
|ATLANTIC NORTH|188|
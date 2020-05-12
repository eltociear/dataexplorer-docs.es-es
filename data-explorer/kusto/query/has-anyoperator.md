---
title: 'operador de has_any: Explorador de datos de Azure'
description: En este artículo se describe has_any operador en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 19329b8822a1e1d484c5f751f5fbc2f8eb6343ac
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226744"
---
# <a name="has_any-operator"></a>Operador has_any

`has_any`operadores filtros basados en el conjunto de valores proporcionado.

```kusto
Table1 | where col has_any ('value1', 'value2')
```

**Sintaxis**

*T* `|` `where` *col* `has_any` `(` *Lista de columnas de expresiones escalares*`)`   
*T* `|` `where` *col* `has_any` `(` *Expresión tabular* T col`)`   
 
**Argumentos**

* Entrada de *T* -tabular cuyos registros se van a filtrar.
* columna *col* que se va a filtrar.
* *lista de expresiones* : lista separada por comas de expresiones tabulares, escalares o literales  
* *expresión tabular* : expresión tabular que tiene un conjunto de valores (si la expresión tiene varias columnas, se usa la primera columna)

**Devuelve**

Filas de *T* para las que el predicado es`true`

**Notas**

* La lista de expresiones puede generar valores de hasta `10,000` .    
* En el caso de las expresiones tabulares, se selecciona la primera columna del conjunto de resultados.   

**Ejemplos:**  

**Un uso sencillo del `has_any` operador:**  

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
StormEvents 
| where State has_any ("CAROLINA", "DAKOTA", "NEW") 
| summarize count() by State
```

|Estado|count_|
|---|---|
|NUEVA YORK|1750|
|CAROLINA DEL NORTE|1721|
|DAKOTA DEL SUR|1567|
|NUEVA JERSEY|1044|
|CAROLINA DEL SUR|915|
|DAKOTA DEL NORTE|905|
|NUEVO MÉXICO|527|
|NUEVA HAMPSHIRE|394|


**Usar matriz dinámica:**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let states = dynamic(['south', 'north']);
StormEvents 
| where State has_any (states)
| summarize count() by State
```

|Estado|count_|
|---|---|
|CAROLINA DEL NORTE|1721|
|DAKOTA DEL SUR|1567|
|CAROLINA DEL SUR|915|
|DAKOTA DEL NORTE|905|
|SUR DE ATLÁNTICO|193|
|ATLÁNTICO NORTE|188|

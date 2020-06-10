---
title: materializar ()-Azure Explorador de datos
description: En este artículo se describe la función materializar () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 06/06/2020
ms.openlocfilehash: f5ea896d4701aa5aec1b22c1ff20853aea18f065
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84664949"
---
# <a name="materialize"></a>materialize()

Permite almacenar en caché el resultado de una subconsulta durante el tiempo de ejecución de la consulta de forma que otras subconsultas puedan hacer referencia al resultado parcial.
 
**Sintaxis**

`materialize(`*expression*`)`

**Argumentos**

* *expresión*: expresión tabular que se va a evaluar y almacenar en la memoria caché durante la ejecución de la consulta.

**Sugerencias**

* Use materializar con join o Union cuando sus operandos tienen subconsultas mutuas que se pueden ejecutar una vez. Consulte los ejemplos siguientes.

* También resulta útil en escenarios en los que necesita usar operadores join/union para las bifurcaciones.

* Materializar solo se puede usar en instrucciones Let si se asigna un nombre al resultado almacenado en caché.

**Note**

* Materializar tiene un límite de tamaño de caché de **5 GB**. 
  Este límite es por nodo de clúster y es mutua para todas las consultas que se ejecutan simultáneamente.
  Si una consulta utiliza `materialize()` y la memoria caché no puede contener más datos, la consulta se anulará con un error.

**Ejemplos**

En el ejemplo siguiente se muestra cómo `materialize()` se puede usar para mejorar el rendimiento de la consulta.
La expresión `_detailed_data` se define con la `materialize()` función y, por tanto, se calcula solo una vez.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let _detailed_data = materialize(StormEvents | summarize Events=count() by State, EventType);
_detailed_data
| summarize TotalStateEvents=sum(Events) by State
| join (_detailed_data) on State
| extend EventPercentage = Events*100.0 / TotalStateEvents
| project State, EventType, EventPercentage, Events
| top 10 by EventPercentage
```

|Estado|EventType|EventPercentage|Eventos|
|---|---|---|---|
|AGUAS HAWAIS|Waterspout|100|2|
|LAKE ONTARIO|Viento tormenta Marina|100|8|
|GOLFO DE ALASKA|Waterspout|100|4|
|ATLÁNTICO NORTE|Viento tormenta Marina|95.2127659574468|179|
|ERIE DE LAKE|Viento tormenta Marina|92.5925925925926|25|
|E PACÍFICO|Waterspout|90|9|
|LAGO DE MICHIGAN|Viento tormenta Marina|85.1648351648352|155|
|HURON DE LAKE|Viento tormenta Marina|79.3650793650794|50|
|GOLFO DE MÉXICO|Viento tormenta Marina|71.7504332755633|414|
|Hawái|Gran navegación|70.0218818380744|320|


En el ejemplo siguiente se genera un conjunto de números aleatorios y se calcula: 
* cuántos valores distintos hay en el conjunto (DCont)
* los tres valores principales del conjunto 
* la suma de todos estos valores en el conjunto 
 
Esta operación se puede realizar con [lotes](batches.md) y materializar:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let randomSet = 
    materialize(
        range x from 1 to 3000000 step 1
        | project value = rand(10000000));
randomSet | summarize Dcount=dcount(value);
randomSet | top 3 by value;
randomSet | summarize Sum=sum(value)
```

Conjunto de resultados 1:  

|DCount|
|---|
|2578351|

Conjunto de resultados 2: 

|value|
|---|
|9999998|
|9999998|
|9999997|

Conjunto de resultados 3: 

|Sum|
|---|
|15002960543563|

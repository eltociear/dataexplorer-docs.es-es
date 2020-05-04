---
title: materializar ()-Azure Explorador de datos
description: En este artículo se describe la materialización () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/21/2019
ms.openlocfilehash: e721e5809d3b0445fecc0609668332b66ef39db8
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737341"
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


* Materializar tiene un límite de tamaño de caché de **5 GB**. 
  Este límite es por nodo de clúster y es mutua para todas las consultas que se ejecutan simultáneamente.
  Si una consulta utiliza `materialize()` y la memoria caché no puede contener más datos, la consulta se anulará con un error.

**Ejemplos**

Queremos generar un conjunto aleatorio de valores y desea saber: 
 * cuántos valores distintos tenemos 
 * la suma de todos estos valores 
 * los tres valores principales

Esta operación se puede realizar con [lotes](batches.md) y materializar:

 ```kusto
let randomSet = materialize(range x from 1 to 30000000 step 1
| project value = rand(10000000));
randomSet
| summarize dcount(value);
randomSet
| top 3 by value;
randomSet
| summarize sum(value)

```
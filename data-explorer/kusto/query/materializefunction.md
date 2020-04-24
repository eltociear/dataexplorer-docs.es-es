---
title: materialize() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe materialize() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/21/2019
ms.openlocfilehash: dfaed8cf972a517c86717999b3e5423c0ddd1fb0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512621"
---
# <a name="materialize"></a>materialize()

Permite almacenar en caché un resultado de subconsulta durante el tiempo de ejecución de la consulta de forma que otras subconsultas puedan hacer referencia al resultado parcial.

 
**Sintaxis**

`materialize(`*Expresión*`)`

**Argumentos**

* *expresión*: Expresión tabular que se evaluará y almacenará en caché durante la ejecución de la consulta.

**Sugerencias**

* Use materialize cuando tenga operadores join/union en los que sus operandos tengan subconsultas mutuas que se pueden ejecutar una vez (consulte los ejemplos a continuación).

* También resulta útil en escenarios en los que necesita usar operadores join/union para las bifurcaciones.

* Materialize solo se puede usar en instrucciones let si se asigna un nombre al resultado en caché.

* Materialize tiene un límite de tamaño de caché de **5 GB.** 
  Este límite es por nodo de clúster y es mutuo para todas las consultas que se ejecutan simultáneamente.
  Si se `materialize()` utiliza una consulta y la memoria caché no puede contener datos adicionales, la consulta se anula con un error.

**Ejemplos**

Suponiendo que queremos generar un conjunto aleatorio de valores y estamos interesados en encontrar cuánto valores distintos tenemos, la suma de todos estos valores y los 3 valores principales.

Esto se puede hacer usando [lotes](batches.md) y materializar:

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
---
title: toscalar() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe toscalar() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 60fe8123760a9921bfa7abfacbbdffba6dba8d7b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505906"
---
# <a name="toscalar"></a>toscalar()

Devuelve un valor constante escalar de la expresión evaluada. 

Esta función es útil para las consultas que requieren cálculos por etapas, como por ejemplo calcular un recuento total de eventos y, a continuación, usarla para los grupos de filtrado que superan cierto porcentaje de todos los eventos. 

**Sintaxis**

`toscalar(`*Expresión*`)`

**Argumentos**

* *Expresión*: Expresión que se evaluará para la conversión escalar  

**Devuelve**

Un valor constante escalar de la expresión evaluada.
Si el resultado de la expresión es un tabular, se tomará la primera columna y la primera fila para la conversión.

> [!TIP]
> Puede usar una [instrucción let](letstatement.md) para legibilidad de la consulta al usar `toscalar()`.

**Notas**

`toscalar()`se puede calcular un número constante de veces durante la ejecución de la consulta.
En otras `toscalar()` palabras, la función no se puede aplicar en el nivel de fila de (para cada escenario de fila).

**Ejemplos**

La siguiente consulta `Start` `End` evalúa `Step` y como constantes escalares `range` y la usa para la evaluación. 

```kusto
let Start = toscalar(print x=1);
let End = toscalar(range x from 1 to 9 step 1 | count);
let Step = toscalar(2);
range z from Start to End step Step | extend start=Start, end=End, step=Step
```

|z|start|end|paso|
|---|---|---|---|
|1|1|9|2|
|3|1|9|2|
|5|1|9|2|
|7|1|9|2|
|9|1|9|2|
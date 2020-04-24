---
title: 'operador de invocación: Explorador de datos de Azure . . . . . . . . . . . . . Microsoft Docs'
description: En este artículo se describe el operador invoke en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 41f19440795f4f302352a8dda5192c5c4790ea99
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513709"
---
# <a name="invoke-operator"></a>Operador invoke

Invoca lambda que recibe el `invoke` origen de como argumento de parámetro tabular.

```kusto
T | invoke foo(param1, param2)
```

**Sintaxis**

`T | invoke`*función*`(`[*param1* `,` *param2*]`)`

**Argumentos**

* *T*: La fuente tabular.
* *función*: el nombre de la expresión lambda o el nombre de la función que se va a evaluar.
* *param1*, *param2* ... : argumentos lambda adicionales.

**Devuelve**

Devuelve el resultado de la expresión evaluada.

**Notas**

Consulte let instrucciones para obtener más [detalles](./letstatement.md) sobre cómo declarar expresiones lambda que pueden aceptar argumentos tabulares.

**Ejemplo**

En el ejemplo siguiente `invoke` se muestra cómo utilizar operator para llamar a la expresión lambda:

```kusto
// clipped_average(): calculates percentiles limits, and then makes another 
//                    pass over the data to calculate average with values inside the percentiles
let clipped_average = (T:(x: long), lowPercentile:double, upPercentile:double)
{
   let high = toscalar(T | summarize percentiles(x, upPercentile));
   let low = toscalar(T | summarize percentiles(x, lowPercentile));
   T 
   | where x > low and x < high
   | summarize avg(x) 
};
range x from 1 to 100 step 1
| invoke clipped_average(5, 99)
```

|avg_x|
|---|
|52|

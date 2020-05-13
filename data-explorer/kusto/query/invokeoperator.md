---
title: 'operador de invocación: Azure Explorador de datos'
description: En este artículo se describe el operador de invocación en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1aca8cda34e1ee8506d5be6633cfd46fd912c6c3
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271526"
---
# <a name="invoke-operator"></a>Operador invoke

Invoca una expresión lambda que recibe el origen de `invoke` como argumento de parámetro tabular.

```kusto
T | invoke foo(param1, param2)
```

**Sintaxis**

`T | invoke`*función* `(` [*param1* `,` *parám2*]`)`

**Argumentos**

* *T*: el origen tabular.
* *función*: el nombre de la expresión lambda o el nombre de la función que se va a evaluar.
* *parám1*, *parám2* ...: argumentos lambda adicionales.

**Devuelve**

Devuelve el resultado de la expresión evaluada.

**Notas**

Vea [instrucciones Let](./letstatement.md) para obtener más detalles sobre cómo declarar expresiones lambda que pueden aceptar argumentos tabulares.

**Ejemplo**

En el ejemplo siguiente se muestra cómo usar el `invoke` operador para llamar a una expresión lambda:

<!-- csl: https://help.kusto.windows.net:443/KustoMonitoringPersistentDatabase -->
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

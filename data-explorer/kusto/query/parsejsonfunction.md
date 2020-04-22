---
title: parse_json() - Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe parse_json() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f87c2155864f039fb5b261786d0fa6eb6770af2f
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744679"
---
# <a name="parse_json"></a>parse_json()

Interpreta `string` un valor JSON como y `dynamic`devuelve el valor como . 

Es superior al uso de la [función extractjson()](./extractjsonfunction.md) cuando necesita extraer más de un elemento de un objeto compuesto JSON.

**Sintaxis**

`parse_json(`*Json*`)`

Alias:
- [todynamic()](./todynamicfunction.md)
- [toobject()](./todynamicfunction.md)

**Argumentos**

* *json*: Una `string`expresión de tipo , que representa un valor con formato `dynamic` [JSON](https://json.org/)o una expresión de tipo [dynamic](./scalar-data-types/dynamic.md), que representa el valor real.

**Devuelve**

Objeto de `dynamic` tipo determinado por el valor de *json:*
* Si *json* es `dynamic`de tipo , su valor se utiliza tal como es.
* Si *json* es `string`de tipo , y es una [cadena JSON con el formato correcto,](https://json.org/)se analiza la cadena y se devuelve el valor generado.
* Si *json* es `string`de tipo , pero **no** es una cadena JSON con el [formato correcto,](https://json.org/)el valor devuelto es un objeto de tipo `dynamic` que contiene el valor original. `string`

**Ejemplo**

En el ejemplo siguiente, cuando `context_custom_metrics` es un elemento `string` similar a este: 

```json
{"duration":{"value":118.0,"count":5.0,"min":100.0,"max":150.0,"stdDev":0.0,"sampledValue":118.0,"sum":118.0}}
```

a continuación, el siguiente fragmento de `duration` CSL recupera el valor de la `duration.value` ranura `duration.min` `118.0` en `110.0`el objeto y, a partir de eso, recupera dos ranuras y , respectivamente).

```kusto
T
| extend d=parse_json(context_custom_metrics) 
| extend duration_value=d.duration.value, duration_min=d["duration"]["min"]
```

**Notas**

Es algo común tener una cadena JSON que describa un contenedor de propiedades en el que una de las "ranuras" es otra cadena JSON. Por ejemplo:

```kusto
let d='{"a":123, "b":"{\\"c\\":456}"}';
print d
```

En tales casos, no sólo `parse_json` es necesario invocar dos veces, sino `tostring` también para asegurarse de que en la segunda llamada, se utilizará. De lo contrario, `parse_json` la segunda llamada simplemente pasará la entrada a la `dynamic`salida tal como está, porque su tipo declarado es:

```kusto
let d='{"a":123, "b":"{\\"c\\":456}"}';
print d_b_c=parse_json(tostring(parse_json(d).b)).c
```
---
title: 'El tipo de datos de intervalo de tiempo: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el tipo de datos timespan en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 31a0bfafed817ffaf531cffdcb844da8a357531f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509612"
---
# <a name="the-timespan-data-type"></a>El tipo de datos de intervalo de tiempo

El `timespan` `time`tipo de datos ( ) representa un intervalo de tiempo.

## <a name="timespan-literals"></a>literales de lapso de tiempo

Los literales `timespan` de `timespan(`tipo tienen el *valor*`)`de sintaxis, donde se admiten varios formatos para *value*, como se indica en la tabla siguiente:

|||
---|---
`2d`|2 días
`1.5h`|1,5 horas
`30m`|30 minutos
`10s`|10 segundos
`0.1s`|0,1 segundos
`100ms`| 100 milisegundos
`10microsecond`|10 microsegundos
`1tick`|100 ns
`time(15 seconds)`|15 segundos
`time(2)`| 2 días
`time(0.12:34:56.7)`|`0d+12h+34m+56.7s`

El formulario `time(null)` especial es el [valor nulo](null-values.md).

## <a name="timespan-operators"></a>operadores de intervalo de tiempo

Se pueden `timespan` agregar, restar y dividir dos valores de tipo.
La última operación devuelve `real` un valor de tipo que representa el número fraccionario de veces que un valor puede ajustarse al otro.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se calcula cuántos segundos hay en un día de varias maneras:

```kusto
print
    result1 = 1d / 1s,
    result2 = time(1d) / time(1s),
    result3 = 24 * 60 * time(00:01:00) / time(1s)
```
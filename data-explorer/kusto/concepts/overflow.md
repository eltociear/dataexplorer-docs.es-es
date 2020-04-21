---
title: 'Desbordamientos: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los desbordamientos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 22db905788e1313cad2eb229239a390c28bcd5c2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523161"
---
# <a name="overflows"></a>Desbordamientos

Un desbordamiento se produce cuando el resultado de un cálculo es demasiado grande para el tipo de destino.
Este fenómeno suele dar lugar a un error parcial de [consulta.](partialqueryfailures.md)

Por ejemplo, la siguiente consulta dará lugar a un desbordamiento:

```kusto
let Weight = 92233720368547758;
range x from 1 to 3 step 1
| summarize percentilesw(x, Weight * 100, 50)
```

La implementación `percentilesw()` de Kusto acumula la `Weight` expresión para los valores que están "lo suficientemente cerca".
En este caso, la acumulación desencadena un desbordamiento porque no cabe en un entero de 64 bits con signo.

Normalmente, sin embargo, los desbordamientos son el resultado de un "bug" en la consulta, ya que Kusto utiliza tipos de 64 bits para cálculos aritméticos.
El mejor curso de acción en estos casos es identificar desde el mensaje de error qué función o agregación desencadenó un desbordamiento y asegurarse de que sus argumentos de entrada se evalúan como valores que tienen sentido.

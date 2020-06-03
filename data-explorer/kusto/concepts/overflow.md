---
title: 'Desbordamientos: Azure Explorador de datos'
description: En este artículo se describen los desbordamientos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 063165c91319ef5f4183a8ce8f83364fd8188072
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84328966"
---
# <a name="overflows"></a>Desbordamientos

Un desbordamiento se produce cuando el resultado de un cálculo es demasiado grande para el tipo de destino.
El desbordamiento normalmente conduce a un [error de consulta parcial](partialqueryfailures.md).

Por ejemplo, la siguiente consulta producirá un desbordamiento.

```kusto
let Weight = 92233720368547758;
range x from 1 to 3 step 1
| summarize percentilesw(x, Weight * 100, 50)
```

`percentilesw()`La implementación de Kusto acumula la `Weight` expresión para los valores que son "suficientemente cercanos".
En este caso, la acumulación desencadena un desbordamiento porque no cabe en un entero de 64 bits con signo.

Normalmente, los desbordamientos son el resultado de un "error" en la consulta, ya que Kusto usa tipos de 64 bits para cálculos aritméticos.
La mejor opción es examinar el mensaje de error e identificar la función o agregación que desencadenó el desbordamiento. Asegúrese de que los argumentos de entrada se evalúan como valores que tienen sentido.

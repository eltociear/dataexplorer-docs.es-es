---
title: 'array_slice (): Explorador de datos de Azure'
description: En este artículo se describe array_slice () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/03/2018
ms.openlocfilehash: 98ed69475232dde474f21d1f4a7f792c82a69637
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225622"
---
# <a name="array_slice"></a>array_slice()

Extrae un segmento de una matriz dinámica.

**Sintaxis**

`array_slice`(*`arr`*, *`start`*, *`end`*])

**Argumentos**

* *`arr`*: La matriz de entrada de la que se va a extraer el segmento debe ser dinámica.
* *`start`*: índice de inicio de base cero (incluido) del segmento; los valores negativos se convierten en array_length + Start.
* *`end`*: índice final de base cero (inclusivo) del segmento; los valores negativos se convierten en array_length + final.

Nota: se omiten los índices fuera de los límites.

**Devuelve**

Matriz dinámica de los valores del intervalo [ `start..end` ] de `arr` .

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic([1,2,3]) 
| extend sliced=array_slice(arr, 1, 2)
```
|`arr`|`sliced`|
|---|---|
|[1, 2, 3]|[2,3]|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend sliced=array_slice(arr, 2, -1)
```
|`arr`|segmentar|
|---|---|
|[1, 2, 3, 4, 5]|[3, 4, 5]|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic([1,2,3,4,5]) 
| extend sliced=array_slice(arr, -3, -2)
```
|`arr`|segmentar|
|---|---|
|[1, 2, 3, 4, 5]|[3,4]|

---
title: prev() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe prev() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 33f6045333826b21ddc0092e2cc7d5e033c12a96
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744601"
---
# <a name="prev"></a>prev()

Devuelve el valor de una columna de una fila que se encuentra en algún desplazamiento anterior a la fila actual en un conjunto de [filas serializado.](./windowsfunctions.md#serialized-row-set)

**Sintaxis**

`prev(column)`

`prev(column, offset)`

`prev(column, offset, default_value)`

**Argumentos**

* `column`: la columna de la que se obtienen los valores.

* `offset`: el desplazamiento para volver en filas. Cuando no se especifica ningún desplazamiento, se utiliza un desplazamiento predeterminado 1.

* `default_value`: el valor predeterminado que se utilizará cuando no haya filas anteriores de las que tomar el valor. Cuando no se especifica ningún valor predeterminado, se utiliza null.


**Ejemplos**

```kusto
Table | serialize | extend prevA = prev(A,1)
| extend diff = A - prevA
| where diff > 1

Table | serialize prevA = prev(A,1,10)
| extend diff = A - prevA
| where diff <= 10
```
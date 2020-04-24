---
title: round() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe round() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 05111b6261c14f21d8d08e88a4c070faab32dc4c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510241"
---
# <a name="round"></a>round()

Devuelve el origen redondeado a la precisión especificada.

**Sintaxis**

`round(`*fuente* `,` [ *Precisión*]`)`

**Argumentos**

* *fuente*: Se calcula el escalar de origen sobre el que se calcula la ronda.
* *Precisión*: Número de dígitos a los que se redondeará la fuente. (el valor predeterminado es 0)

**Devuelve**

El origen redondeado a la precisión especificada.

La ronda [`bin()`](binfunction.md) / [`floor()`](floorfunction.md) es diferente de que la primera redondea un número a un número específico de dígitos, mientras que el valor de las últimas rondas a un múltiplo entero de un tamaño de bin determinado (round(2.15, 1) devuelve 2.2 mientras que bin(2.15, 1) devuelve 2).
 

**Ejemplos**

```kusto
round(2.15, 1)                   // 2.2
round(2.15) (which is the same as round(2.15, 0))                   // 2
round(-50.55, -2)                   // -100
round(21.5, -1)                   // 20
```
---
title: binary_all_xor() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe binary_all_xor() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/06/2020
ms.openlocfilehash: a1908fe874576281c9ba45f23709845473b5725c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517704"
---
# <a name="binary_all_xor-aggregation-function"></a>binary_all_xor() (función de agregación)

Acumula valores utilizando `XOR` la operación binaria por grupo de integración (o en total, si la integración se realiza sin agrupación).

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `binary_all_xor(` *Expr*`)`

**Argumentos**

* *Expr*: número largo.

**Devuelve**

Devuelve un valor que se `XOR` agrega mediante la operación binaria sobre los registros por grupo de integración (o en total, si la integración se realiza sin agrupación).

**Ejemplo**

Producción de 'café-alimento' mediante operaciones binarias: `XOR`

```kusto
datatable(num:long)
[
  0x44404440,
  0x1E1E1E1E,
  0x90ABBA09,
  0x000B105A,
]
| summarize result = toupper(tohex(binary_all_xor(num)))
```

|resultado|
|---|
|CAFEF00D|
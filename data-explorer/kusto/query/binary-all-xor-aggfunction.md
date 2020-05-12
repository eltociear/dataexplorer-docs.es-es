---
title: 'binary_all_xor () (función de agregación): Explorador de datos de Azure'
description: En este artículo se describe binary_all_xor () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/06/2020
ms.openlocfilehash: bc4b0bc8a02dd3a8d2a39ffdd27db5817eb8ffdb
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225248"
---
# <a name="binary_all_xor-aggregation-function"></a>binary_all_xor () (función de agregación)

Acumula valores mediante la operación binaria `XOR` por grupo de resumen (o en total, si el resumen se realiza sin agrupación).

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

resumir `binary_all_xor(` *expr*`)`

**Argumentos**

* *Expr*: número largo.

**Devuelve**

Devuelve un valor que se agrega mediante la operación binaria `XOR` sobre registros por grupo de resumen (o en total, si el resumen se realiza sin agrupación).

**Ejemplo**

Generar "café-Food" mediante operaciones binarias `XOR` :

<!-- csl: https://help.kusto.windows.net/Samples -->
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

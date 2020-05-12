---
title: 'binary_all_and () (función de agregación): Explorador de datos de Azure'
description: En este artículo se describe binary_all_and () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 9f0e1665010885a64e6d97151b074d3a03df829b
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227577"
---
# <a name="binary_all_and-aggregation-function"></a>binary_all_and () (función de agregación)

Acumula valores mediante la operación binaria `AND` por grupo de resumen (o en total, si el resumen se realiza sin agrupación).

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

resumir `binary_all_and(` *expr*`)`

**Argumentos**

* *Expr*: número largo.

**Devuelve**

Devuelve un valor que se agrega mediante la operación binaria `AND` sobre registros por grupo de resumen (o en total, si el resumen se realiza sin agrupación).

**Ejemplo**

Generar "café-Food" mediante operaciones binarias `AND` :

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(num:long)
[
  0xFFFFFFFF, 
  0xFFFFF00F,
  0xCFFFFFFD,
  0xFAFEFFFF,
]
| summarize result = toupper(tohex(binary_all_and(num)))
```

|resultado|
|---|
|CAFEF00D|

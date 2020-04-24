---
title: binary_all_or() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe binary_all_or() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 5de240f606e53b26996ebfe11073170758233e2c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517721"
---
# <a name="binary_all_or-aggregation-function"></a>binary_all_or() (función de agregación)

Acumula valores utilizando `OR` la operación binaria por grupo de integración (o en total, si la integración se realiza sin agrupación).

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `binary_all_or(` *Expr*`)`

**Argumentos**

* *Expr*: número largo.

**Devuelve**

Devuelve un valor que se `OR` agrega mediante la operación binaria sobre los registros por grupo de integración (o en total, si la integración se realiza sin agrupación).

**Ejemplo**

Producción de 'café-alimento' mediante operaciones binarias: `OR`

```kusto
datatable(num:long)
[
  0x88888008,
  0x42000000,
  0x00767000,
  0x00000005, 
]
| summarize result = toupper(tohex(binary_all_or(num)))
```

|resultado|
|---|
|CAFEF00D|
---
title: binary_all_and() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe binary_all_and() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 4dfe4a2881f100a4bea3e9d418022c75b2621087
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517755"
---
# <a name="binary_all_and-aggregation-function"></a>binary_all_and() (función de agregación)

Acumula valores utilizando `AND` la operación binaria por grupo de integración (o en total, si la integración se realiza sin agrupación).

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `binary_all_and(` *Expr*`)`

**Argumentos**

* *Expr*: número largo.

**Devuelve**

Devuelve un valor que se `AND` agrega mediante la operación binaria sobre los registros por grupo de integración (o en total, si la integración se realiza sin agrupación).

**Ejemplo**

Producción de 'café-alimento' mediante operaciones binarias: `AND`

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
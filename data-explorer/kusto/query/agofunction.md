---
title: ago() - Explorador de Azure Data Explorer - Azure Data Explorer - Microsoft Docs
description: En este artículo se describe ago() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7d155f1a1cd113f73acc779e6c2e73d5f537e71c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519098"
---
# <a name="ago"></a>ago()

Resta un intervalo de tiempo especificado a la hora UTC actual.

```kusto
ago(1h)
ago(1d)
```

Al igual que `now()`, esta función se puede usar varias veces en una instrucción, y la hora UTC a la que se hace referencia será la misma para todas las instancias.

**Sintaxis**

`ago(`*a_timespan*`)`

**Argumentos**

* *a_timespan*: intervalo para restar a la hora UTC actual (`now()`).

**Devuelve**

`now() - a_timespan`

**Ejemplo**

Todas las filas con una marca de tiempo de la última hora:

```kusto
T | where Timestamp > ago(1h)
```
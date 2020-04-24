---
title: now() - Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe now() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c1a130cfbd45c35ff1ba26ed6c47986fc186c89c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512060"
---
# <a name="now"></a>now()

Devuelve la hora de reloj UTC actual, opcionalmente compensada por un intervalo de tiempo determinado.
Esta función puede utilizarse varias veces en una instrucción, y la hora UTC a la que se hace referencia será la misma para todas las instancias.

```kusto
now()
now(-2d)
```

**Sintaxis**

`now(`[*desplazamiento*]`)`

**Argumentos**

* *desplazamiento*: `timespan`A , añadido a la hora actual del reloj UTC. Valor predeterminado: 0.

**Devuelve**

La hora UTC actual como `datetime`.

`now()` + *Compensar* 

**Ejemplo**

Determina el intervalo desde el evento identificado por el predicado:

```kusto
T | where ... | extend Elapsed=now() - Timestamp
```
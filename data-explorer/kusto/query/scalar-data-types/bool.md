---
title: El tipo de datos bool - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el tipo de datos bool en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: dd9cfa4f97f3baa4353f967f5e1ca31186f4815f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509935"
---
# <a name="the-bool-data-type"></a>El tipo de datos bool

El `bool` `boolean`tipo de datos ( ) `true` puede `false` tener uno `1` `0`de dos estados: o (codificado internamente como y , respectivamente), así como el valor nulo.

## <a name="bool-literals"></a>bool literales

El `bool` tipo de datos tiene los siguientes literales:
* `true`y `bool(true)`: Representando la verdad
* `false`y `bool(false)`: Representar la falsedad
* `bool(null)`: Ver [valores nulos](null-values.md)

## <a name="bool-operators"></a>operadores de bool

El `bool` tipo de datos admite los`==`siguientes operadores: equality ( ), inequality (`!=`), logical-and (`and`), y logical-or (`or`).
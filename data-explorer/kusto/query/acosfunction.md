---
title: acos() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe acos() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: aa33714d57b319ba5a775385e8ee7d232addfe9d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519336"
---
# <a name="acos"></a>acos()

Devuelve el ángulo cuyo coseno es el número especificado [`cos()`](cosfunction.md)(la operación inversa de ) .

**Sintaxis**

`acos(`*X*`)`

**Argumentos**

* *x*: Un número real en el rango [-1, 1].

**Devuelve**

* El valor del coseno de arco de`x`
* `null`si `x` < -1 o `x` > 1
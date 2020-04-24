---
title: log2() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe log2() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 41e9a1457f97fa04a4daa54e1929f27d8a448ae3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513131"
---
# <a name="log2"></a>log2()

`log2()`devuelve la función de logarithm base-2.  

**Sintaxis**

`log2(`*X*`)`

**Argumentos**

* *x*: Un número real > 0.

**Devuelve**

* El laquearitmo es el laqueiritmo base-2: el inverso de la función exponencial (exp) con la base 2.
* `null`si el argumento es negativo o null o `real` no se puede convertir en un valor. 

**Vea también**

* Para logaritmos naturales (base-e), véase [log()](log-function.md).
* Para los logaritmos comunes (base-10), véase [log10()](log10-function.md).
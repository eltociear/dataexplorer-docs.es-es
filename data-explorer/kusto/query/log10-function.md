---
title: log10() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe log10() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 9ccc83ff466d0414f793b7cfbbcf10d2ca169348
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513199"
---
# <a name="log10"></a>log10()

`log10()`devuelve la función de laritmo común (base-10).  

**Sintaxis**

`log10(`*X*`)`

**Argumentos**

* *x*: Un número real > 0.

**Devuelve**

* El laqueiritmo común es el laqueiritmo base-10: el inverso de la función exponencial (exp) con la base 10.
* `null`si el argumento es negativo o null o `real` no se puede convertir en un valor. 

**Vea también**

* Para logaritmos naturales (base-e), véase [log()](log-function.md).
* Para logaritmos base-2, véase [log2()](log2-function.md)
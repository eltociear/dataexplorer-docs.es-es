---
title: log() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe log() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 7bd3c4f5c6b262587cab2327486945f5d78aaf02
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513284"
---
# <a name="log"></a>log()

`log()`devuelve la función de logartmhm natural.  

**Sintaxis**

`log(`*X*`)`

**Argumentos**

* *x*: Un número real > 0.

**Devuelve**

* El laquearitmo natural es el laquearitmo base-e: el inverso de la función exponencial natural (exp).
* `null`si el argumento es negativo o null o `real` no se puede convertir en un valor. 

**Vea también**

* Para los logaritmos comunes (base-10), véase [log10()](log10-function.md).
* Para logaritmos base-2, véase [log2()](log2-function.md)
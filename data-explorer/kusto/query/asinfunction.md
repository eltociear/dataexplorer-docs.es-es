---
title: asin() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe asin() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 5db112daeba59dd841b02df8ba1a41185654ad6a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518554"
---
# <a name="asin"></a>asin()

Devuelve el ángulo cuyo seno es el número especificado [`sin()`](sinfunction.md)(la operación inversa de ) .

**Sintaxis**

`asin(`*X*`)`

**Argumentos**

* *x*: Un número real en el rango [-1, 1].

**Devuelve**

* El valor del seno de arco de`x`
* `null`si `x` < -1 o `x` > 1
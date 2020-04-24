---
title: sqrt() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe sqrt() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 660235a60893732288a551e1febd9b7b044b4f00
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507266"
---
# <a name="sqrt"></a>sqrt()

Devuelve la función de raíz cuadrada.  

**Sintaxis**

`sqrt(`*X*`)`

**Argumentos**

* *x*: Un número real >0.

**Devuelve**

* Un número positivo, como `sqrt(x) * sqrt(x) == x`
* `null` si el argumento es negativo o no se puede convertir a un valor de `real`. 
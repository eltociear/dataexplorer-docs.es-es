---
title: El tipo de datos real- Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe el tipo de datos real en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: f69b74670fefcff6f6c24d5235f58beb98b0405a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509680"
---
# <a name="the-real-data-type"></a>El tipo de datos real

El `real` tipo de datos representa un número de punto flotante de doble precisión de 64 bits de ancho.

Los literales `real` del tipo de datos tienen la misma representación que . NET's `System.Double`. `1.0`, `0.1`, `1e5` y son todos `real`literales de tipo .

Hay varias formas literales especiales:
* `real(null)`: Es el [valor nulo](null-values.md).
* `real(nan)`: No-a-Number (NaN). Por ejemplo, el resultado `0.0` de `0.0`dividir a por otro .
* `real(+inf)`: Infinito positivo. Por ejemplo, el resultado `1.0` `0.0`de dividir por .
* `real(-inf)`: Infinito negativo. Por ejemplo, el resultado `-1.0` `0.0`de dividir por .
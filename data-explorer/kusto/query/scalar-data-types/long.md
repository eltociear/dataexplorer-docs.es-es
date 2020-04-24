---
title: El tipo de datos largo- Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe el tipo de datos long en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: b8c51f216b8515e483daf64d9783210ff35ceef3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509714"
---
# <a name="the-long-data-type"></a>El tipo de datos largo

El `long` tipo de datos representa un entero con signo de 64 bits de ancho.

## <a name="long-literals"></a>literales largos

Los literales `long` del tipo de datos se pueden especificar en la sintaxis siguiente:

`long``(` *Valor*`)`

Donde *Value* puede adoptar las siguientes formas:
* Uno más o dígitos, en cuyo caso el valor literal es la representación decimal de estos dígitos. Por ejemplo, `long(12)` es el `long`número doce de tipo .
* El `0x` prefijo seguido de uno o más dígitos hexadecimales. Por ejemplo, `long(0xf)` es equivalente a `long(15)`.
* Un signo`-`menos ( ) seguido de uno o más dígitos. Por ejemplo, `long(-1)` es el número `long`menos uno de tipo .
* `null`, en cuyo caso es el `long` valor [nulo](null-values.md) del tipo de datos. Por lo tanto, `long` el `long(null)`valor nulo de type es .

Kusto también admite literales `long` `long(` / `)` de tipo sin el prefijo/suffi solo para las dos primeras formas. Por `123` lo tanto, `long`es un `0x123`literal `-2` de tipo , como es , pero `-` **no** es un literal (actualmente se interpreta como la función unaria aplicada al literal `2` de tipo long).
 
Para convertir long en hex string - vea [la función tohex()](../tohexfunction.md).
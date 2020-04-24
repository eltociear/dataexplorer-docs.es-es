---
title: max_of() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe max_of() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 68188ccd5eb814a22be166b8847d80193172813f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512502"
---
# <a name="max_of"></a>max_of()

Devuelve el valor máximo de varias expresiones numéricas evaluadas.

```kusto
max_of(10, 1, -3, 17) == 17
```

**Sintaxis**

`max_of``(` *expr_1* expr_1`,` *expr_2* ...`)`

**Argumentos**

* *expr_i:* Una expresión escalar que se va a evaluar.

- Todos los argumentos deben ser del mismo tipo.
- Se admite un máximo de 64 argumentos.

**Devuelve**

El valor máximo de todas las expresiones de argumento.

**Ejemplo**

```kusto
print result = max_of(10, 1, -3, 17) 
```

|resultado|
|---|
|17|
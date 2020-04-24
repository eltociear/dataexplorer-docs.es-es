---
title: min_of() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe min_of() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 06a8f7ce6bcef8f3c15c4ea3d4c997b4e4540bf7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512400"
---
# <a name="min_of"></a>min_of()

Devuelve el valor mínimo de varias expresiones numéricas evaluadas.

```kusto
min_of(10, 1, -3, 17) == -3
```

**Sintaxis**

`min_of``(` *expr_1* expr_1`,` *expr_2* ...`)`

**Argumentos**

* *expr_i:* Una expresión escalar que se va a evaluar.

- Todos los argumentos deben ser del mismo tipo.
- Se admite un máximo de 64 argumentos.

**Devuelve**

El valor mínimo de todas las expresiones de argumento.

**Ejemplo**

```kusto
print result=min_of(10, 1, -3, 17) 
```

|resultado|
|---|
|-3|
---
title: 'max_of (): Explorador de datos de Azure'
description: En este artículo se describe max_of () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4b912b1bdc68d7b3071ace8547f0aaf7c679a86a
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271611"
---
# <a name="max_of"></a>max_of()

Devuelve el valor máximo de varias expresiones numéricas evaluadas.

```kusto
max_of(10, 1, -3, 17) == 17
```

**Sintaxis**

`max_of``(` *expr_1* `,` *expr_2* ...`)`

**Argumentos**

* *expr_i*: expresión escalar que se va a evaluar.

- Todos los argumentos deben ser del mismo tipo.
- Se admite un máximo de 64 argumentos.

**Devuelve**

Valor máximo de todas las expresiones de argumento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples  -->
```kusto
print result = max_of(10, 1, -3, 17) 
```

|resultado|
|---|
|17|

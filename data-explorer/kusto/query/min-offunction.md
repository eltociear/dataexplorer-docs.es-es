---
title: 'min_of (): Explorador de datos de Azure'
description: En este artículo se describe min_of () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0762b1416df32279b9801c47f129a6966772a7e2
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271373"
---
# <a name="min_of"></a>min_of()

Devuelve el valor mínimo de varias expresiones numéricas evaluadas.

```kusto
min_of(10, 1, -3, 17) == -3
```

**Sintaxis**

`min_of``(` *expr_1* `,` *expr_2* ...`)`

**Argumentos**

* *expr_i*: expresión escalar que se va a evaluar.

- Todos los argumentos deben ser del mismo tipo.
- Se admite un máximo de 64 argumentos.

**Devuelve**

El valor mínimo de todas las expresiones de argumento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples  -->
```kusto
print result=min_of(10, 1, -3, 17) 
```

|resultado|
|---|
|-3|

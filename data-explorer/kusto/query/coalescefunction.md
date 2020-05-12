---
title: 'Coalesce (): Azure Explorador de datos'
description: En este artículo se describe COALESCE () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ea57efe36fb86189d798e5f18fa3fe9470bfd634
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227543"
---
# <a name="coalesce"></a>coalesce()

Evalúa una lista de expresiones y devuelve la primera expresión que no es null (o no está vacía para la cadena).

```kusto
coalesce(tolong("not a number"), tolong("42"), 33) == 42
```

**Sintaxis**

`coalesce(`*expr_1* `, ` *expr_2* `,` ...)

**Argumentos**

* *expr_i*: expresión escalar que se va a evaluar.
- Todos los argumentos deben ser del mismo tipo.
- Se admite un máximo de 64 argumentos.


**Devuelve**

Valor de la primera *expr_i* cuyo valor no es null (o no está vacío para las expresiones de cadena).

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples  -->
```kusto
print result=coalesce(tolong("not a number"), tolong("42"), 33)
```

|resultado|
|---|
|42|

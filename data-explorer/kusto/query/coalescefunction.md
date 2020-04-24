---
title: coalesce() - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe coalesce() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1ee2cd24f36007914fdc326e2863da148aec4406
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517143"
---
# <a name="coalesce"></a>coalesce()

Evalúa una lista de expresiones y devuelve la primera expresión no nula (o no vacía para cadena).

```kusto
coalesce(tolong("not a number"), tolong("42"), 33) == 42
```

**Sintaxis**

`coalesce(`*expr_1*`, `*expr_2* `,` ...)

**Argumentos**

* *expr_i:* Una expresión escalar que se va a evaluar.
- Todos los argumentos deben ser del mismo tipo.
- Se admite un máximo de 64 argumentos.


**Devuelve**

El valor de la primera *expr_i* cuyo valor no es null (o no vacío para las expresiones de cadena).

**Ejemplo**

```kusto
print result=coalesce(tolong("not a number"), tolong("42"), 33)
```

|resultado|
|---|
|42|
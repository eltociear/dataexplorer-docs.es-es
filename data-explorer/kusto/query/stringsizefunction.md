---
title: string_size() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe string_size() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7886e7b8fd5039c9b197ae435bce4f40b93e5038
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506892"
---
# <a name="string_size"></a>string_size()

Devuelve el tamaño, en bytes, de la cadena de entrada.

**Sintaxis**

`string_size(`*Fuente*`)`

**Argumentos**

* *source*: La cadena de origen que se medirá para el tamaño de la cadena.

**Devuelve**

Devuelve la longitud, en bytes, de la cadena de entrada.

**Ejemplos**

```kusto
print size = string_size("hello")
```

|tamaño|
|---|
|5|

```kusto
print size = string_size("⒦⒰⒮⒯⒪")
```

|tamaño|
|---|
|15|
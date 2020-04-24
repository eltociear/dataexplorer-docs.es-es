---
title: extract() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe extract() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 398c79ddcd362f3c09fea18accd082c0eb3b1fc3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515375"
---
# <a name="extract"></a>extract()

Obtenga una coincidencia para una [expresión regular](./re2.md) a partir de una cadena de texto. 

Opcionalmente, convierta la subcadena extraída al tipo indicado.

    extract("x=([0-9.]+)", 1, "hello x=45.6|wo") == "45.6"

**Sintaxis**

`extract(`*regex* `,` *captureTexto* `,` `,` de *grupo* [ *typeLiteral*]`)`

**Argumentos**

* *regex*: Una [expresión regular](./re2.md).
* *captureGroup*: `int` Una constante positiva que indica el grupo de captura que se ha de extraer. 0 significa toda la coincidencia; 1, el valor que coincide con el primer elemento "("paréntesis")" de la expresión regular; 2 o más, los posteriores paréntesis.
* *texto*: `string` A para buscar.
* *typeLiteral*: Un literal de tipo `typeof(long)`opcional (por ejemplo, ). Si se proporciona, la subcadena extraída se convierte a este tipo. 

**Devuelve**

Si *regex* encuentra una coincidencia en *text*: la subcadena coincidía con el grupo de captura indicado, *captureGroup*, opcionalmente convertido a *typeLiteral*.

Si no hay ninguna coincidencia, o se produce un error en la conversión del tipo: `null`. 

**Ejemplos**

En la cadena de ejemplo `Trace` se busca una definición para `Duration`. La coincidencia se convierte en `real` y se multiplica por una constante de tiempo (`1s`) para que `Duration` sea de tipo `timespan`. En este ejemplo, es igual a 123,45 segundos:

```kusto
...
| extend Trace="A=1, B=2, Duration=123.45, ..."
| extend Duration = extract("Duration=([0-9.]+)", 1, Trace, typeof(real)) * time(1s) 
```

En este ejemplo, es equivalente a `substring(Text, 2, 4)`:

```kusto
extract("^.{2,2}(.{4,4})", 1, Text)
```
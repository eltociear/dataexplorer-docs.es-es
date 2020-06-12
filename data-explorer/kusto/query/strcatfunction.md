---
title: 'strcat (): Azure Explorador de datos'
description: En este artículo se describe strcat () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: af25bb0407c9bc0c004c2f22e326ac034c6682cd
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717111"
---
# <a name="strcat"></a>strcat()

Concatena entre 1 y 64 argumentos.

* Si los argumentos no son de tipo cadena, se convertirán forzosamente en una cadena.

**Sintaxis**

`strcat(`*argumento1*, *argument2*[, *argumentn*]`)`

**Argumentos**

* *argumento1* ... *argumentn*: expresiones que se van a concatenar.

**Devuelve**

Argumentos, concatenados en una sola cadena.

**Ejemplos**
  
   ```kusto
print str = strcat("hello", " ", "world")
```

|str|
|---|
|hola a todos|

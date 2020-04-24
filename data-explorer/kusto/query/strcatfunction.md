---
title: strcat() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe strcat() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: dd01f875b45be038371cc184987aa2a8f8b8d5eb
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506926"
---
# <a name="strcat"></a>strcat()

Concatena entre 1 y 64 argumentos.

* En caso de que los argumentos no sean de tipo cadena, se convertirán forzosamente en cadena.

**Sintaxis**

`strcat(`*argument1*,*argument2* [, *argumentN*]`)`

**Argumentos**

* *argument1* ... *argumentN* : expresiones que se van a concatenar.

**Devuelve**

Argumentos concatenados en una sola cadena.

**Ejemplos**
  
   ```kusto
print str = strcat("hello", " ", "world")
```

|str|
|---|
|hola a todos|
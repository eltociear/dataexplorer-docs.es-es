---
title: strcat_delim() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe strcat_delim() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f944af741cd5f655c2c9b090ddebc6cc35a47766
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506943"
---
# <a name="strcat_delim"></a>strcat_delim()

Concatena entre 2 y 64 argumentos, con delimitador, proporcionado como primer argumento.

 * En caso de que los argumentos no sean de tipo cadena, se convertirán forzosamente en cadena.

**Sintaxis**

`strcat_delim(`*delimitador*,*argument1*,*argument2* [, *argumentN*]`)`

**Argumentos**

* *delimitador*: expresión de cadena, que se utilizará como separador.
* *argument1* ... *argumentN* : expresiones que se van a concatenar.

**Devuelve**

Argumentos concatenados en una sola cadena con *delimitador*.

**Ejemplos**

```kusto
print st = strcat_delim('-', 1, '2', 'A', 1s)

```

|st|
|---|
|1-2-A-00:00:01|
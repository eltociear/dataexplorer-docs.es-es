---
title: assert() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe assert() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: 97f01df7bc85171eefabeb2ed835109f0faef81e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518418"
---
# <a name="assert"></a>assert()

Comprueba una condición. Si la condición es false, genera mensajes de error y produce un error en la consulta.

**Sintaxis**

`assert(`*condition*`, `*mensaje de* condición`)`

**Argumentos**

* *condición*: la expresión condicional que se va a evaluar. Si la `false`condición es , el mensaje especificado se utiliza para notificar un error. Si la `true`condición `true` es , vuelve como resultado de la evaluación. La condición debe evaluarse como constante durante la fase de análisis de consultas.
* *message*: El mensaje utilizado si `false`la aserción se evalúa como . El *mensaje* debe ser un literal de cadena.


**Devuelve**

* `true`- si la afección es`true`
* Genera un error semántico si `false`la condición se evalúa como .

**Notas**

* `condition`debe evaluarse como constante durante la fase de análisis de consultas. En otras palabras, se puede construir a partir de otras expresiones que hacen referencia a constantes y no se puede enlazar al contexto de fila.

**Ejemplos**

La siguiente consulta `checkLength()` define una función que `assert` comprueba la longitud de la cadena de entrada y se usa para validar el parámetro de longitud de entrada (comprueba que es mayor que cero).

```kusto
let checkLength = (len:long, s:string)
{
    assert(len > 0, "Length must be greater than zero") and 
    strlen(s) > len
};
datatable(input:string)
[
    '123',
    '4567'
]
| where checkLength(len=long(-1), input)
```

La ejecución de esta consulta produce un error:  
`assert() has failed with message: 'Length must be greater than zero'`


Ejemplo de ejecución `len` con entrada válida:

```kusto
let checkLength = (len:long, s:string)
{
    assert(len > 0, "Length must be greater than zero") and strlen(s) > len
};
datatable(input:string)
[
    '123',
    '4567'
]
| where checkLength(len=3, input)
```

|input|
|---|
|4567|
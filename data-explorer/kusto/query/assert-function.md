---
title: Assert ()-Azure Explorador de datos
description: En este artículo se describe Assert () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: b1f83f0b78e4bbb16de706a8d14ca04ee522c2ee
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225570"
---
# <a name="assert"></a>assert()

Comprueba una condición. Si la condición es falsa, genera mensajes de error y la consulta no se realiza correctamente.

**Sintaxis**

`assert(`*condición* `, ` de *mensaje* de`)`

**Argumentos**

* *condición*: expresión condicional que se va a evaluar. Si la condición es `false` , el mensaje especificado se utiliza para informar de un error. Si la condición es `true` , se devuelve `true` como resultado de la evaluación. La condición debe evaluarse como constante durante la fase de análisis de consulta.
* *Message*: el mensaje que se usa si la aserción se evalúa como `false` . El *mensaje* debe ser un literal de cadena.


**Devuelve**

* `true`: Si la condición es`true`
* Genera un error semántico si la condición se evalúa como `false` .

**Notas**

* `condition`se debe evaluar como constante durante la fase de análisis de consulta. En otras palabras, se puede construir a partir de otras expresiones que hacen referencia a constantes y no se pueden enlazar a un contexto de fila.

**Ejemplos**

En la consulta siguiente se define una función `checkLength()` que comprueba la longitud de la cadena de entrada y utiliza `assert` para validar el parámetro de longitud de entrada (comprueba que es mayor que cero).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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


Ejemplo de ejecución con una `len` entrada válida:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

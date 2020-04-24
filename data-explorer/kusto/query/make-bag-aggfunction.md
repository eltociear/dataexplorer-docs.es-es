---
title: make_bag() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_bag() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 16f5f5663c4807a766d99c12020ff0a46c4db336
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512995"
---
# <a name="make_bag-aggregation-function"></a>make_bag() (función de agregación)

Devuelve `dynamic` un (JSON) property-bag (dictionary) de todos los valores de *Expr* en el grupo.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_bag(` *Expr* `,` [ *MaxSize*]`)`

**Argumentos**

* *Expr*: Expresión `dynamic` de tipo que se utilizará para el cálculo de la agregación.
* *MaxSize* es un límite entero opcional en el número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

**Nota**

Una variante heredada y obsoleta `make_dictionary()` de esta función: tiene un límite predeterminado de *MaxSize* 128.

**Devuelve**

Devuelve `dynamic` un (JSON) property-bag (dictionary) de todos los valores de *Expr* en el grupo que son property-bags (diccionarios).
Se omitirán los valores que no sean diccionarios.
Si una clave aparece en más de una fila, se elegirá un valor arbitrario (de los valores posibles para esta clave).

**Vea también**

Utilice el complemento [bag_unpack()](bag-unpackplugin.md) para expandir objetos JSON dinámicos en columnas mediante claves de contenedor de propiedades. 

**Ejemplos**

```kusto
let T = datatable(prop:string, value:string)
[
    "prop01", "val_a",
    "prop02", "val_b",
    "prop03", "val_c",
];
T
| extend p = pack(prop, value)
| summarize dict=make_bag(p)

```

|dict|
|----|
|" prop01": "val_a", "prop02": "val_b", "prop03": "val_c" |

Utilice el plugin [bag_unpack()](bag-unpackplugin.md) para transformar las teclas de la bolsa en la salida make_bag() en columnas. 

```kusto
let T = datatable(prop:string, value:string)
[
    "prop01", "val_a",
    "prop02", "val_b",
    "prop03", "val_c",
];
T
| extend p = pack(prop, value)
| summarize bag=make_bag(p)
| evaluate bag_unpack(bag) 

```

|prop01|prop02|prop03|
|---|---|---|
|val_a|val_b|val_c|
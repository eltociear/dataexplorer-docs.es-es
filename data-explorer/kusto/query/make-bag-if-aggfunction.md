---
title: make_bag_if() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_bag_if() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 201de637df387bb8925995a5e52c048255d1535c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512978"
---
# <a name="make_bag_if-aggregation-function"></a>make_bag_if() (función de agregación)

Devuelve `dynamic` un (JSON) property-bag (dictionary) de todos los valores de *Expr* en el grupo, para el que *Predicate* se evalúa como `true`.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_bag_if(` *Expr*,`,` *Predicado* [ *MaxSize*]`)`

**Argumentos**

* *Expr*: Expresión `dynamic` de tipo que se utilizará para el cálculo de la agregación.
* *Predicado*: Predicado que tiene que evaluarse como `true`, para que *Expr* se agregue al resultado.
* *MaxSize* es un límite entero opcional en el número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

**Devuelve**

Devuelve `dynamic` un (JSON) property-bag (dictionary) de todos los valores de *Expr* en el grupo que `true`son property-bags (diccionarios), para el que *Predicate* se evalúa como .
Se omitirán los valores que no sean diccionarios.
Si una clave aparece en más de una fila, se elegirá un valor arbitrario (de los valores posibles para esta clave).

**Vea también**

[`make_bag`](./make-bag-aggfunction.md)función, que hace lo mismo, sin expresión de predicado.

**Ejemplos**

```kusto
let T = datatable(prop:string, value:string, predicate:bool)
[
    "prop01", "val_a", true,
    "prop02", "val_b", false,
    "prop03", "val_c", true
];
T
| extend p = pack(prop, value)
| summarize dict=make_bag_if(p, predicate)

```

|dict|
|----|
|" prop01": "val_a", "prop03": "val_c" |

Utilice el plugin [bag_unpack()](bag-unpackplugin.md) para transformar las teclas de la bolsa en la salida make_bag_if() en columnas. 

```kusto
let T = datatable(prop:string, value:string, predicate:bool)
[
    "prop01", "val_a", true,
    "prop02", "val_b", false,
    "prop03", "val_c", true
];
T
| extend p = pack(prop, value)
| summarize bag=make_bag_if(p, predicate)
| evaluate bag_unpack(bag) 

```

|prop01|prop03|
|---|---|
|val_a|val_c|
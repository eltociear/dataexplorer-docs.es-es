---
title: buildschema() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe buildschema() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: d0faceae970034ae96ced2b8958af4f16cb5dff4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517296"
---
# <a name="buildschema-aggregation-function"></a>buildschema() (función de agregación)

Devuelve el esquema mínimo que admite todos los valores de *DynamicExpr*.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `buildschema(` *DynamicExpr*`)`

**Argumentos**

* *DynamicExpr*: Expresión que se utilizará para el cálculo de la agregación. El tipo de `dynamic`columna de parámetro debe ser . 

**Devuelve**

El valor máximo de *Expr* en todo el grupo.

> [!TIP] 
> Si `buildschema(json_column)` da un error de sintaxis: *¿Es `json_column` la cadena en lugar de un objeto dinámico?* Si es así, `buildschema(parsejson(json_column))`debe utilizar .

**Ejemplo**

Suponga que la columna de entrada tiene tres valores dinámicos:

||
|---|
|`{"x":1, "y":3.5}`|
|`{"x":"somevalue", "z":[1, 2, 3]}`|
|`{"y":{"w":"zzz"}, "t":["aa", "bb"], "z":["foo"]}`|
||

El esquema resultante sería:

    { 
      "x":["int", "string"], 
      "y":["double", {"w": "string"}], 
      "z":{"`indexer`": ["int", "string"]}, 
      "t":{"`indexer`": "string"} 
    }

El esquema indica que:

* El objeto raíz es un contenedor con cuatro propiedades denominadas x, y, z y t.
* La propiedad denominada "x" podría ser de tipo "int" o de tipo "string".
* La propiedad llamada "y" podría del tipo "double", u otro contenedor con una propiedad llamada "w" de tipo "string".
* La palabra clave ``indexer`` indica que "z" y "t" son matrices.
* Cada elemento de la matriz "z" es un entero o una cadena.
* "t" es una matriz de cadenas.
* Cada propiedad es implícitamente opcional, y cualquier matriz puede estar vacía.

### <a name="schema-model"></a>Modelo de esquema

La sintaxis del esquema devuelto es:

    Container ::= '{' Named-type* '}';
    Named-type ::= (name | '"`indexer`"') ':' Type;
    Type ::= Primitive-type | Union-type | Container;
    Union-type ::= '[' Type* ']';
    Primitive-type ::= "int" | "string" | ...;

Son equivalentes a un subconjunto de las anotaciones de tipo TypeScript, codificadas como un valor dinámico de Kusto. En Typescript, el esquema de ejemplo sería:

    var someobject: 
    { 
      x?: (number | string), 
      y?: (number | { w?: string}), 
      z?: { [n:number] : (int | string)},
      t?: { [n:number]: string } 
    }
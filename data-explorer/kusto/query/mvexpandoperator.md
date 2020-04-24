---
title: operador mv-expand - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe el operador mv-expand en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2019
ms.openlocfilehash: 01a4b56155d1c29727670b4e827cb7b9968ef7a9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512298"
---
# <a name="mv-expand-operator"></a>Operador mv-expand

Expande la matriz de varios valores o la bolsa de propiedades.

`mv-expand`se aplica en una columna con tipo [dinámico](./scalar-data-types/dynamic.md)para que cada valor de la colección obtenga una fila independiente. Todas las demás columnas de una fila expandida se duplican. 

**Sintaxis**

*T* `| mv-expand ` `bagexpansion=`[`bag` | `array`(`with_itemindex=`)] [*IndexColumnName*] *ColumnName* [`,` *ColumnName* ...] [`limit` *Límite de filas*]

*T* `| mv-expand ` `bagexpansion=`[`bag` | `array`( )] [`to typeof(`*Name* `=`] *ArrayExpression* [*Typename*`)`] [, [*Name* `=`] *ArrayExpression* [`to typeof(`*Typename*`)`] ...] [`limit` *Límite de filas*]

**Argumentos**

* *ColumnName:* en el resultado, las matrices en la columna indicada se expanden a varias filas. 
* *ArrayExpression:* expresión que produce una matriz. Si se usa esta forma, se agrega una nueva columna y se conserva la existente.
* *Name:* un nombre para la nueva columna.
* *Nombre de tipo:* Indica el tipo subyacente de los elementos de la matriz, que se convierte en el tipo de la columna producida por el operador.
    Tenga en cuenta que los valores de la matriz que no se ajustan a este tipo no se convertirán; más bien, tomarán `null` un valor.
* *RowLimit:* el número máximo de filas generadas a partir de cada fila original. El valor predeterminado es 2147483647. 
*Nota:* La forma heredada `mvexpand` y obsoleta del operador tiene un límite de fila predeterminado de 128.
* *IndexColumnName:* Si `with_itemindex` se especifica, la salida incluirá una columna adicional (denominada *IndexColumnName*), que contiene el índice (a partir de 0) del elemento de la colección expandida original. 

**Devuelve**

Varias filas para cada uno de los valores de una matriz en la columna indicada o en la expresión de matriz.
Si se especifican varias columnas o expresiones, se expanden en paralelo, por lo que para cada fila de entrada habrá tantas filas de salida como elementos en la expresión expandida más larga (las listas más cortas se rellenan con valores NULL). Si el valor de una fila es una matriz vacía, la fila se expande a nada (no se mostrará en el conjunto de resultados). Si el valor de una fila no es una matriz, la fila se mantiene tal como está en el conjunto de resultados. 

La columna expandida siempre tiene el tipo dinámico. Use una conversión como `todatetime()` o `tolong()` si desea calcular o agregar valores.

Se admiten dos modos de expansiones de contenedor de propiedades:
* `bagexpansion=bag`: los contenedores de propiedades se expanden en contenedores de propiedades de entrada única. Esta es la expansión predeterminada.
* `bagexpansion=array`: Las bolsas de propiedades `[`se expanden en estructuras de matriz de*valores* `]` clave de dos *elementos,*`,`lo que permite un acceso uniforme a claves y valores (así como, por ejemplo, ejecutar una agregación de recuento distinto sobre los nombres de propiedad). 

**Ejemplos**

Una simple expansión de una sola columna:
 ```kusto
datatable (a:int, b:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"})]
| mv-expand b 
```

|a|b|
|---|---|
|1|"prop1":"a"|
|1|"prop2":"b"|


La expansión de dos columnas primero 'zip' las columnas aplicables y luego las expandirá:

```kusto
datatable (a:int, b:dynamic, c:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"}), dynamic([5])]
| mv-expand b, c 
```

|a|b|c|
|---|---|---|
|1|"prop1":"a"|5|
|1|"prop2":"b"||

Si desea obtener un producto cartesiano de expandir dos columnas, expanda una tras otra:
```kusto
datatable (a:int, b:dynamic, c:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"}), dynamic([5])]
| mv-expand b 
| mv-expand c
```

|a|b|c|
|---|---|---|
|1|"prop1":"a"|5|
|1|"prop2":"b"|5|


Expansión de `with_itemindex`una matriz con:
```kusto
range x from 1 to 4 step 1 
| summarize x = make_list(x) 
| mv-expand with_itemindex=Index  x 
```

|x|Índice|
|---|---|
|1|0|
|2|1|
|3|2|
|4|3|


**Más ejemplos**

Consulte [Recuento de gráficos de actividades en vivo](./samples.md#concurrent-activities)a lo largo del tiempo.

**Vea también**

- [operador mv-apply.](./mv-applyoperator.md)
- [resumir make_list()](makelist-aggfunction.md) que realiza la función opuesta.
- [bag_unpack()](bag-unpackplugin.md) plugin para expandir objetos JSON dinámicos en columnas utilizando claves de bolsa de propiedades.
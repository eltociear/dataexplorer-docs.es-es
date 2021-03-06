---
title: 'operador MV-Expand: Azure Explorador de datos'
description: En este artículo se describe el operador MV-Expand en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2019
ms.openlocfilehash: bba3c4e82c3c1ac53a6ebafcb9de4c327da77e37
ms.sourcegitcommit: 4986354cc1ba25c584e4f3c7eac7b5ff499f0cf1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84856181"
---
# <a name="mv-expand-operator"></a>Operador mv-expand

Expande la matriz de varios valores o el contenedor de propiedades.

`mv-expand`se aplica en una columna de tipo [dinámico](./scalar-data-types/dynamic.md)para que cada valor de la colección obtenga una fila independiente. Todas las demás columnas de una fila expandida se duplican. 

**Sintaxis**

*T* `| mv-expand ` [ `bagexpansion=` ( `bag`  |  `array` )] [ `with_itemindex=` *IndexColumnName*] *columnName* [ `,` *columnName* ...] [ `limit` *ROWLIMIT*]

*T* `| mv-expand ` [ `bagexpansion=` ( `bag`  |  `array` )] [*nombre* `=` ] *ArrayExpression* [ `to typeof(` *TypeName* `)` ] [, [*nombre* `=` ] *ArrayExpression* [ `to typeof(` *TypeName* `)` ]...] [ `limit` *ROWLIMIT*]

**Argumentos**

* *ColumnName:* en el resultado, las matrices en la columna indicada se expanden a varias filas. 
* *ArrayExpression:* expresión que produce una matriz. Si se usa esta forma, se agrega una nueva columna y se conserva la existente.
* *Name:* un nombre para la nueva columna.
* *TypeName:* Indica el tipo subyacente de los elementos de la matriz, que se convierte en el tipo de la columna generada por el operador. Los valores no conformes de la matriz no se convertirán. En su lugar, estos valores tomarán un `null` valor.
* *RowLimit:* el número máximo de filas generadas a partir de cada fila original. El valor predeterminado es 2147483647. 
  > [!Note] 
  > La forma heredada y obsoleta del operador `mvexpand` tiene un límite de filas predeterminado de 128.
* *IndexColumnName:* Si `with_itemindex` se especifica, el resultado incluirá una columna adicional (denominada *IndexColumnName*), que contiene el índice (a partir de 0) del elemento de la colección expandida original. 

**Devuelve**

Varias filas para cada uno de los valores de una matriz que se encuentran en la columna con nombre o en la expresión de matriz.
Si se especifican varias columnas o expresiones, se expanden en paralelo. Para cada fila de entrada, habrá tantas filas de salida como elementos haya en la expresión expandida más larga (las listas más cortas se rellenan con valores NULL). Si el valor de una fila es una matriz vacía, la fila se expande a nada (no se mostrará en el conjunto de resultados). Sin embargo, si el valor de una fila no es una matriz, la fila se mantiene tal cual en el conjunto de resultados. 

La columna expandida siempre tiene el tipo dinámico. Use una conversión como `todatetime()` o `tolong()` si desea calcular o agregar valores.

Se admiten dos modos de expansiones de contenedor de propiedades:
* `bagexpansion=bag`: los contenedores de propiedades se expanden en contenedores de propiedades de entrada única. Este modo es la expansión predeterminada.
* `bagexpansion=array`: Los contenedores de propiedades se expanden en estructuras de matriz de valores de clave de dos elementos `[` *key* `,` *value* `]` , lo que permite el acceso uniforme a claves y valores (también, por ejemplo, la ejecución de una agregación de recuento distinto de los nombres de propiedad). 

**Ejemplos**

Una simple expansión de una sola columna:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
 ```kusto
datatable (a:int, b:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"})]
| mv-expand b 
```

|a|b|
|---|---|
|1|{"Prop1": "a"}|
|1|{"Prop2": "b"}|

Al expandir dos columnas, primero se "zip" las columnas aplicables y, a continuación, se expanden:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
datatable (a:int, b:dynamic, c:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"}), dynamic([5])]
| mv-expand b, c 
```

|a|b|c|
|---|---|---|
|1|{"Prop1": "a"}|5|
|1|{"Prop2": "b"}||

Si desea obtener un producto cartesiano de expandir dos columnas, expanda una después de la otra:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
datatable (a:int, b:dynamic, c:dynamic)[1,dynamic({"prop1":"a", "prop2":"b"}), dynamic([5])]
| mv-expand b 
| mv-expand c
```

|a|b|c|
|---|---|---|
|1|{"Prop1": "a"}|5|
|1|{"Prop2": "b"}|5|


Expansión de una matriz con `with_itemindex` :

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 4 step 1
| summarize x = make_list(x)
| mv-expand with_itemindex=Index x
```

|x|Índice|
|---|---|
|1|0|
|2|1|
|3|2|
|4|3|


**Más ejemplos**

Consulte [recuento de gráficos de actividades en directo a lo largo del tiempo](./samples.md#chart-concurrent-sessions-over-time).

**Vea también**

- operador [MV-Apply](./mv-applyoperator.md) .
- [resumir make_list ()](makelist-aggfunction.md), que realiza la función opuesta.
- complemento de [bag_unpack ()](bag-unpackplugin.md) para expandir objetos JSON dinámicos en columnas mediante claves de contenedor de propiedades.

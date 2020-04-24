---
title: El tipo de datos dinámicos- Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el tipo de datos dinámicos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: e1cdb6e5af20b326198a7447c50c24e5f632d237
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509884"
---
# <a name="the-dynamic-data-type"></a>El tipo de datos dinámicos

El `dynamic` tipo de datos escalares es especial, ya que puede asumir cualquier valor de otros tipos de datos escalares de la lista siguiente, así como matrices y bolsas de propiedades. Específicamente, `dynamic` un valor puede ser:

* Null.
* Valor de cualquiera de los tipos de `bool` `datetime`datos `guid` `int`escalares `real` `string`primitivos: , , , , `long`, , , y `timespan`.
* Matriz de `dynamic` valores, con cero o más valores con indexación de base cero.
* Un contenedor de `string` propiedades `dynamic` que asigna valores únicos a valores.
  El contenedor de propiedades tiene cero o más asignaciones de `string` este tipo (denominadas "ranuras"), indizadas por los valores únicos. Las ranuras están desordenadas.

> [!NOTE]
> Los valores `dynamic` de tipo están limitados a 1 MB (2 x 20).

> [!NOTE]
> Aunque `dynamic` el tipo aparece como JSON, puede contener valores que el modelo JSON no representa porque `long`no `real` `datetime`existen `timespan`en `guid`JSON (por ejemplo, , , , y ).
> Por lo tanto, al serializar `dynamic` valores en una representación JSON, los valores que JSON no puede representar se serializan en `string` valores. Por el contrario, Kusto analizará las cadenas como valores fuertemente tipados si se pueden analizar como tales.
> Esto se `datetime` `real`aplica `long`a `guid` los tipos , , , y . Para obtener más información sobre el modelo de objetos JSON, consulte [json.org](https://json.org/).

> [!NOTE]
> Kusto no intenta conservar el orden de las asignaciones de nombre a valor en una bolsa de propiedades, por lo que no puede asumir el orden que se va a conservar. Es totalmente posible que dos bolsas de propiedades con el mismo conjunto de `string` asignaciones produzcan resultados diferentes cuando se representan como valores, por ejemplo.

## <a name="dynamic-literals"></a>Literales dinámicos

Un literal `dynamic` de tipo tiene este aspecto:

`dynamic(`*Valor*`)`

*El valor* puede ser:

* `null`, en cuyo caso el literal representa `dynamic(null)`el valor dinámico nulo: .
* Otro literal de tipo de datos escalares, `dynamic` en cuyo caso el literal representa el literal del tipo "interior". Por ejemplo, `dynamic(4)` es un valor dinámico que contiene el valor 4 del tipo de datos escalar largo.
* Matriz de literales dinámicos `[` u otros: *ListOfValues* `]`. Por ejemplo, `dynamic([1, 2, "hello"])` es una matriz dinámica `long` de `string` tres elementos, dos valores y un valor.
* Una bolsa `{` de propiedades: *Valor de* *nombre* `=` ... `}`. Por ejemplo, `dynamic({"a":1, "b":{"a":2}})` es una bolsa `a`de `b`propiedades con dos ranuras, y , con la segunda ranura otra bolsa de propiedades.

```kusto
print o=dynamic({"a":123, "b":"hello", "c":[1,2,3], "d":{}})
| extend a=o.a, b=o.b, c=o.c, d=o.d
```

Para mayor `dynamic` comodidad, los literales que aparecen en el propio texto `datetime` de `timespan` consulta también pueden incluir otros literales de Kusto (como literales, literales, etc.) Esta extensión a través de JSON no está disponible `parse_json` al analizar cadenas (por ejemplo, cuando se utiliza la función o al ingiriera datos), pero le permite hacer esto:

```kusto
print d=dynamic({"a": datetime(1970-05-11)})
```

Para analizar `string` un valor que sigue las `dynamic` reglas de `parse_json` codificación JSON en un valor, utilice la función. Por ejemplo:

* `parse_json('[43, 21, 65]')` : Una matriz de números.
* `parse_json('{"name":"Alan", "age":21, "address":{"street":432,"postcode":"JLK32P"}}')`- un diccionario
* `parse_json('21')` : Un valor único de tipo dinámico que contiene un número.
* `parse_json('"21"')` : Un valor único de tipo dinámico que contiene una cadena.
* `parse_json('{"a":123, "b":"hello", "c":[1,2,3], "d":{}}')`- da el `o` mismo valor que en el ejemplo anterior.

> [!NOTE]
> A diferencia de JavaScript, JSON exige`"`el uso de caracteres de comillas dobles ( ) alrededor de cadenas y nombres de propiedad de bolsa de propiedades.
> Por lo tanto, generalmente es más fácil citar un literal`'`de cadena codificado en JSON mediante un carácter de comillas simples ( ).
  
En el ejemplo siguiente se muestra cómo `dynamic` definir una tabla `datetime` que contiene una columna (así como una columna) y, a continuación, ingerir en ella un único registro. también muestra cómo puede codificar cadenas JSON en archivos CSV:

```kusto
// dynamic is just like any other type:
.create table Logs (Timestamp:datetime, Trace:dynamic)

// Everything between the "[" and "]" is parsed as a CSV line would be:
// 1. Since the JSON string includes double-quotes and commas (two characters
//    that have a special meaning in CSV), we must CSV-quote the entire second field.
// 2. CSV-quoting means adding double-quotes (") at the immediate beginning and end
//    of the field (no spaces allowed before the first double-quote or after the second
//    double-quote!)
// 3. CSV-quoting also means doubling-up every instance of a double-quotes within
//    the contents.
.ingest inline into table Logs
  [2015-01-01,"{""EventType"":""Demo"", ""EventValue"":""Double-quote love!""}"]
```

|Timestamp                   | Seguimiento                                                 |
|----------------------------|-------------------------------------------------------|
|2015-01-01 00:00:00.0000000 | "EventType":"Demo","EventValue":"Amor de comillas dobles!"|

## <a name="dynamic-object-accessors"></a>Descriptores de acceso de objetos dinámicos

Para subíndice de un diccionario,`dict.key`utilice la notación`dict["key"]`de puntos ( ) o la notación de corchetes ( ).
Cuando el subíndice es una constante de cadena, ambas opciones son equivalentes.

> [!NOTE] 
> Para utilizar una expresión como subíndice, utilice la notación de corchetes. Cuando se utilizan expresiones aritméticas, la expresión entre corchetes debe ajustarse entre paréntesis.

En los ejemplos siguientes `dict` y `arr` son columnas de tipo dinámico:

|Expression                        | Tipo de expresión de descriptor de acceso | Significado                                                                              | Comentarios                                      |
|----------------------------------|--------------------------|--------------------------------------------------------------------------------------|-----------------------------------------------|
|dict[col]                         | Nombre de entidad (columna)     | Subíndices de un diccionario `col` utilizando los valores de la columna como clave              | La columna debe ser de tipo string                 | 
|arr[índice]                        | Indice de entidad (columna)    | Subíndices de una matriz `index` utilizando los valores de la columna como índice              | La columna debe ser de tipo entero o booleano     | 
|arr[-index]                       | Indice de entidad (columna)    | Recupera el valor 'index'-th del final de la matriz                             | La columna debe ser de tipo entero o booleano     |
|arr[(-1)]                         | Indice de entidad             | Recupera el último valor de la matriz                                                |                                               |
|arr[toint(indexAsString)]         | Llamada a función            | Convierte los valores `indexAsString` de column a int y utilícelos para subíndice de una matriz |                                               |
|dict[['donde']]                   | Palabra clave utilizada como nombre de entidad (columna) | Subíndices de un diccionario `where` utilizando los valores de columna como clave    | Los nombres de entidad que son idénticos a algunas palabras clave de lenguaje de consulta deben citarse | 
|dict.['where'] o dict['where']   | Constante                 | Subíndices de `where` un diccionario utilizando string como clave                              |                                               |

**Consejo de rendimiento:** Prefiere utilizar subíndices constantes siempre que sea posible

El acceso a un `dynamic` subobjeto `dynamic` de un valor produce otro valor, incluso si el subobjeto tiene un tipo subyacente diferente. Utilice `gettype` la función para detectar el tipo subyacente real del valor y cualquiera de la función de conversión que se muestra a continuación para convertirlo en el tipo real.

## <a name="casting-dynamic-objects"></a>Lanzamiento de objetos dinámicos

> Después de subscripting un objeto dinámico, debe convertir el valor a un tipo simple.

|Expression | Valor | Tipo|
|---|---|---|
| X | parse_json('[100,101,102]')| array|
|X[0]|parse_json('100')|dinámico|
|toint(X[1])|101| int|
| Y | parse_json(''a1":100, "a b c":"2015-01-01"')| diccionario|
|Y.a1|parse_json('100')|dinámico|
|Y["a b c"]| parse_json("2015-01-01")|dinámico|
|todate(Y["a b c"])|datetime(2015-01-01)| datetime|

Las funciones de conversión son:

* `tolong()`
* `todouble()`
* `todatetime()`
* `totimespan()`
* `tostring()`
* `toguid()`
* `todynamic()`

## <a name="building-dynamic-objects"></a>Creación de objetos dinámicos

Varias funciones permiten `dynamic` crear nuevos objetos:

* [pack()](../packfunction.md) crea una bolsa de propiedades a partir de pares nombre/valor.
* [pack_array()](../packarrayfunction.md) crea una matriz a partir de pares nombre/valor.
* [range()](../rangefunction.md) crea una matriz con una serie aritmética de números.
* [zip()](../zipfunction.md) empareja los valores "paralelos" de dos matrices en una sola matriz.
* [repeat()](../repeatfunction.md) crea una matriz con un valor repetido.

Además, hay varias funciones de agregado que crean `dynamic` matrices para contener valores agregados:

* [buildschema()](../buildschema-aggfunction.md) devuelve el esquema `dynamic` agregado de varios valores.
* [make_bag()](../make-bag-aggfunction.md) devuelve una bolsa de propiedades de valores dinámicos dentro del grupo.
* [make_bag_if()](../make-bag-if-aggfunction.md) devuelve un contenedor de propiedades de valores dinámicos dentro del grupo (con un predicado).
* [make_list()](../makelist-aggfunction.md) devuelve una matriz que contiene todos los valores, en secuencia.
* [make_list_if()](../makelistif-aggfunction.md) devuelve una matriz que contiene todos los valores, en secuencia (con un predicado).
* [make_list_with_nulls()](../make-list-with-nulls-aggfunction.md) devuelve una matriz que contiene todos los valores, en secuencia, incluidos los valores nulos.
* [make_set()](../makeset-aggfunction.md) devuelve una matriz que contiene todos los valores únicos.
* [make_set_if()](../makesetif-aggfunction.md) devuelve una matriz que contiene todos los valores únicos (con un predicado).

## <a name="operators-and-functions-over-dynamic-types"></a>Operadores y funciones en tipos dinámicos

|||
|---|---|
| *value* `in` *array*| True si hay un elemento de *array* == *value*<br/>`where City in ('London', 'Paris', 'Rome')`
| *value* `!in` *array*| True si no hay ningún elemento de *array* == *value*
|[`array_length(`array`)`](../arraylengthfunction.md)| Es null si no es una matriz
|[`bag_keys(`Bolsa`)`](../bagkeysfunction.md)| Enumera todas las claves raíz de un objeto de bolsa de propiedades dinámico.
|[`extractjson(`path,object`)`](../extractjsonfunction.md)|Usa una ruta para navegar al objeto.
|[`parse_json(`Fuente`)`](../parsejsonfunction.md)| Convierte una cadena JSON en un objeto dinámico.
|[`range(`desde,a,paso`)`](../rangefunction.md)| Una matriz de valores
|[`mv-expand`listColumn](../mvexpandoperator.md) | Replica una fila para cada valor de una lista en una celda especificada.
|[`summarize buildschema(`Columna`)`](../buildschema-aggfunction.md) |Deduce el esquema de tipo del contenido de la columna
|[`summarize make_bag(`Columna`)`](../make-bag-aggfunction.md) | Combina los valores del contenedor de propiedades (diccionario) de la columna en un contenedor de propiedades, sin duplicación de claves.
|[`summarize make_bag_if(`columna,predicado`)`](../make-bag-if-aggfunction.md) | Combina los valores del contenedor de propiedades (diccionario) de la columna en un contenedor de propiedades, sin duplicación de clave (con predicado).
|[`summarize make_list(`columna`)`](../makelist-aggfunction.md)| Acopla grupos de filas y coloca los valores de la columna en una matriz.
|[`summarize make_list_if(`columna,predicado`)`](../makelistif-aggfunction.md)| Aplana grupos de filas y coloca los valores de la columna en una matriz (con predicado).
|[`summarize make_list_with_nulls(`columna`)`](../make-list-with-nulls-aggfunction.md)| Aplana grupos de filas y coloca los valores de la columna en una matriz, incluidos los valores nulos.
|[`summarize make_set(`Columna`)`](../makeset-aggfunction.md) | Acopla grupos de filas y coloca los valores de la columna en una matriz, sin duplicación.
|[`summarize make_bag(`Columna`)`](../make-bag-aggfunction.md) | Combina los valores del contenedor de propiedades (diccionario) de la columna en un contenedor de propiedades, sin duplicación de claves.
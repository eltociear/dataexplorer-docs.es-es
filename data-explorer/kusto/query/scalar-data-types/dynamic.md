---
title: 'El tipo de datos dinámicos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el tipo de datos dinámicos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: 606d48c4bda583ad82404a1b25119ec9beb4c5a9
ms.sourcegitcommit: 6f56b169fda0b74f9569004555a574d8973b1021
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/12/2020
ms.locfileid: "84748937"
---
# <a name="the-dynamic-data-type"></a>El tipo de datos dinámicos

El `dynamic` tipo de datos escalar es especial, ya que puede tomar cualquier valor de otros tipos de datos escalares de la lista siguiente, así como matrices y contenedores de propiedades. En concreto, un `dynamic` valor puede ser:

* Acepta.
* Un valor de cualquiera de los tipos de datos escalares primitivos: `bool` , `datetime` , `guid` , `int` , `long` , `real` , `string` y `timespan` .
* Matriz de `dynamic` valores que contienen cero o más valores con una indización de base cero.
* Contenedor de propiedades que asigna `string` valores únicos a `dynamic` los valores.
  La bolsa de propiedades tiene cero o más asignaciones (denominadas "ranuras"), indizadas por los `string` valores únicos. Las ranuras no están ordenadas.

> [!NOTE]
> * Los valores de tipo `dynamic` están limitados a 1 MB (2 ^ 20).
> * Aunque el `dynamic` tipo aparece como JSON, puede contener valores que el modelo JSON no representa porque no existen en JSON (por ejemplo,,, `long` `real` , `datetime` , `timespan` y `guid` ).
>   Por lo tanto, en la serialización de `dynamic` valores en una representación JSON, los valores que JSON no puede representar se serializan en `string` valores. Por el contrario, Kusto analizará las cadenas como valores fuertemente tipados si se pueden analizar como tales.
>   Esto se aplica a los `datetime` `real` tipos,, `long` y `guid` . 
>   Para obtener más información sobre el modelo de objetos JSON, vea [JSON.org](https://json.org/).
> * Kusto no intenta conservar el orden de las asignaciones de nombre a valor en un contenedor de propiedades, por lo que no se puede asumir el orden que se va a conservar. Es totalmente posible que dos bolsas de propiedades con el mismo conjunto de asignaciones produzcan resultados diferentes cuando se representan como `string` valores, por ejemplo.

## <a name="dynamic-literals"></a>Literales dinámicos

Un literal de tipo `dynamic` tiene el siguiente aspecto:

`dynamic(`*Valor* de`)`

El *valor* puede ser:

* `null`, en cuyo caso el literal representa el valor dinámico NULL: `dynamic(null)` .
* Otro literal de tipo de datos escalar, en cuyo caso el literal representa el `dynamic` literal del tipo "interno". Por ejemplo, `dynamic(4)` es un valor dinámico que contiene el valor 4 del tipo de datos escalar largo.
* Una matriz de valores dinámicos u otros literales: `[` *ListOfValues* `]` . Por ejemplo, `dynamic([1, 2, "hello"])` es una matriz dinámica de tres elementos, dos `long` valores y un `string` valor.
* Un contenedor de propiedades: `{` *nombre* `=` *valor* ... `}` . Por ejemplo, `dynamic({"a":1, "b":{"a":2}})` es un contenedor de propiedades con dos ranuras, `a` y `b` , con la segunda ranura como otro contenedor de propiedades.

```kusto
print o=dynamic({"a":123, "b":"hello", "c":[1,2,3], "d":{}})
| extend a=o.a, b=o.b, c=o.c, d=o.d
```

Para mayor comodidad, `dynamic` los literales que aparecen en el propio texto de consulta también pueden incluir otros literales de Kusto (como `datetime` literales, `timespan` literales, etc.). Esta extensión a través de JSON no está disponible cuando se analizan cadenas (como cuando se usa la `parse_json` función o cuando se ingesta datos), pero se puede hacer esto:

```kusto
print d=dynamic({"a": datetime(1970-05-11)})
```

Para analizar un `string` valor que sigue las reglas de codificación JSON en un `dynamic` valor, utilice la `parse_json` función. Por ejemplo:

* `parse_json('[43, 21, 65]')` : Una matriz de números.
* `parse_json('{"name":"Alan", "age":21, "address":{"street":432,"postcode":"JLK32P"}}')`-un diccionario
* `parse_json('21')` : Un valor único de tipo dinámico que contiene un número.
* `parse_json('"21"')` : Un valor único de tipo dinámico que contiene una cadena.
* `parse_json('{"a":123, "b":"hello", "c":[1,2,3], "d":{}}')`: proporciona el mismo valor que `o` en el ejemplo anterior.

> [!NOTE]
> A diferencia de JavaScript, JSON asigna el uso de caracteres de comillas dobles ( `"` ) alrededor de las cadenas y los nombres de propiedad del contenedor de propiedades.
> Por lo tanto, suele ser más fácil entrecomillar un literal de cadena con codificación JSON mediante el uso de un carácter de comilla simple ( `'` ).
  
En el ejemplo siguiente se muestra cómo puede definir una tabla que contiene una `dynamic` columna (así como una `datetime` columna) y, a continuación, ingerir en ella un solo registro. también muestra cómo puede codificar cadenas JSON en archivos CSV:

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
|2015-01-01 00:00:00.0000000 | {"EventType": "Demo", "EventValue": "Double-quote amort!"}|

## <a name="dynamic-object-accessors"></a>Descriptores de acceso de objetos dinámicos

Para subgenerar un diccionario, use la notación de puntos ( `dict.key` ) o la notación de corchetes ( `dict["key"]` ).
Cuando el subíndice es una constante de cadena, ambas opciones son equivalentes.

> [!NOTE] 
> Para usar una expresión como subíndice, use la notación de corchetes. Al utilizar expresiones aritméticas, la expresión dentro de los corchetes se debe incluir entre paréntesis.

En los ejemplos siguientes `dict` y `arr` son columnas de tipo dinámico:

|Expresión                        | Tipo de expresión accessor | Significado                                                                              | Comentarios                                      |
|----------------------------------|--------------------------|--------------------------------------------------------------------------------------|-----------------------------------------------|
|dict [col]                         | Nombre de entidad (columna)     | Subíndices de un diccionario usando los valores de la columna `col` como clave              | La columna debe ser de tipo cadena                 | 
|ARR [índice]                        | Índice de entidad (columna)    | Subíndices de una matriz usando los valores de la columna `index` como índice              | La columna debe ser de tipo entero o booleano     | 
|ARR [-índice]                       | Índice de entidad (columna)    | Recupera el valor ' index'-th del final de la matriz.                             | La columna debe ser de tipo entero o booleano     |
|ARR [(-1)]                         | Índice de entidad             | Recupera el último valor de la matriz.                                                |                                               |
|ARR [Toint ((indexAsString)]         | Llamada a función            | Convierte los valores de la columna `indexAsString` en int y los usa para subgenerar una matriz. |                                               |
|dict [[' Where ']]                   | Palabra clave usada como nombre de entidad (columna) | Subíndices de un diccionario que usa los valores de la columna `where` como clave    | Los nombres de entidad que son idénticos a algunas palabras clave del lenguaje de consulta deben ir entre comillas | 
|dict. [' Where '] o dict [' Where ']   | Constante                 | Subíndices de un diccionario `where` que usa una cadena como clave                              |                                               |

**Sugerencia de rendimiento:** Preferir el uso de subíndices constantes cuando sea posible

El acceso a un subobjeto de un `dynamic` valor produce otro `dynamic` valor, incluso si el subobjeto tiene un tipo subyacente diferente. Utilice la `gettype` función para detectar el tipo subyacente real del valor y cualquiera de las funciones de conversión que se enumeran a continuación para convertirlo al tipo real.

## <a name="casting-dynamic-objects"></a>Convertir objetos dinámicos

> Después de subgenerar un objeto dinámico, debe convertir el valor en un tipo simple.

|Expresión | Value | Tipo|
|---|---|---|
| X | parse_json (' [100101102] ')| array|
|X [0]|parse_json (' 100 ')|dinámico|
|Toint ((X [1])|101| int|
| Y | parse_json (' {"a1": 100, "a b c": "2015-01-01"} ")| diccionario|
|Y. a1|parse_json (' 100 ')|dinámico|
|Y ["a b c"]| parse_json ("2015-01-01")|dinámico|
|ToDate (Y ["a b c"])|fecha y hora (2015-01-01)| datetime|

Las funciones de conversión son:

* `tolong()`
* `todouble()`
* `todatetime()`
* `totimespan()`
* `tostring()`
* `toguid()`
* `todynamic()`

## <a name="building-dynamic-objects"></a>Compilar objetos dinámicos

Varias funciones permiten crear nuevos `dynamic` objetos:

* [Pack ()](../packfunction.md) crea un contenedor de propiedades a partir de pares nombre-valor.
* [pack_array ()](../packarrayfunction.md) crea una matriz a partir de los pares nombre/valor.
* [Range ()](../rangefunction.md) crea una matriz con una serie aritmética de números.
* [zip ()](../zipfunction.md) empareja los valores "paralelos" de dos matrices en una sola.
* [REPEAT ()](../repeatfunction.md) crea una matriz con un valor repetido.

Además, hay varias funciones de agregado que crean `dynamic` matrices para contener valores agregados:

* [buildschema ()](../buildschema-aggfunction.md) devuelve el esquema agregado de varios `dynamic` valores.
* [make_bag ()](../make-bag-aggfunction.md) devuelve un contenedor de propiedades de valores dinámicos dentro del grupo.
* [make_bag_if ()](../make-bag-if-aggfunction.md) devuelve un contenedor de propiedades de valores dinámicos dentro del grupo (con un predicado).
* [make_list ()](../makelist-aggfunction.md) devuelve una matriz que contiene todos los valores, en secuencia.
* [make_list_if ()](../makelistif-aggfunction.md) devuelve una matriz que contiene todos los valores, en secuencia (con un predicado).
* [make_list_with_nulls ()](../make-list-with-nulls-aggfunction.md) devuelve una matriz que contiene todos los valores, en secuencia, incluidos los valores NULL.
* [make_set ()](../makeset-aggfunction.md) devuelve una matriz que contiene todos los valores únicos.
* [make_set_if ()](../makesetif-aggfunction.md) devuelve una matriz que contiene todos los valores únicos (con un predicado).

## <a name="operators-and-functions-over-dynamic-types"></a>Operadores y funciones en tipos dinámicos

|||
|---|---|
| *value* `in` *array*| True si hay un elemento de *array* == *value*<br/>`where City in ('London', 'Paris', 'Rome')`
| *value* `!in` *array*| True si no hay ningún elemento de *array* == *value*
|[`array_length(`array`)`](../arraylengthfunction.md)| Es null si no es una matriz
|[`bag_keys(`saco`)`](../bagkeysfunction.md)| Enumera todas las claves raíz de un objeto de contenedor de propiedades dinámico.
|[`extractjson(`Ruta de acceso, objeto`)`](../extractjsonfunction.md)|Usa una ruta para navegar al objeto.
|[`parse_json(`fuentes`)`](../parsejsonfunction.md)| Convierte una cadena JSON en un objeto dinámico.
|[`range(`de, de a, paso`)`](../rangefunction.md)| Una matriz de valores
|[`mv-expand`listColumn](../mvexpandoperator.md) | Replica una fila para cada valor de una lista en una celda especificada.
|[`summarize buildschema(`artículo`)`](../buildschema-aggfunction.md) |Deduce el esquema de tipo del contenido de la columna
|[`summarize make_bag(`artículo`)`](../make-bag-aggfunction.md) | Combina los valores del contenedor de propiedades (diccionario) de la columna en un contenedor de propiedades, sin la duplicación de claves.
|[`summarize make_bag_if(`columna, predicado`)`](../make-bag-if-aggfunction.md) | Combina los valores del contenedor de propiedades (diccionario) de la columna en un contenedor de propiedades, sin duplicación de claves (con predicado).
|[`summarize make_list(`columna `)` de](../makelist-aggfunction.md)| Acopla grupos de filas y coloca los valores de la columna en una matriz.
|[`summarize make_list_if(`columna, predicado `)`](../makelistif-aggfunction.md)| Acopla grupos de filas y coloca los valores de la columna en una matriz (con el predicado).
|[`summarize make_list_with_nulls(`columna `)` de](../make-list-with-nulls-aggfunction.md)| Acopla grupos de filas y coloca los valores de la columna en una matriz, incluidos los valores NULL.
|[`summarize make_set(`artículo`)`](../makeset-aggfunction.md) | Acopla grupos de filas y coloca los valores de la columna en una matriz, sin duplicación.
|[`summarize make_bag(`artículo`)`](../make-bag-aggfunction.md) | Combina los valores del contenedor de propiedades (diccionario) de la columna en un contenedor de propiedades, sin la duplicación de claves.

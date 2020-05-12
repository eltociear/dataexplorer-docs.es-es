---
title: 'Funciones definidas por el usuario: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las funciones definidas por el usuario en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: e9e199631d57f3fd3be8d438e6b0cdbf9bdd4266
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227366"
---
# <a name="user-defined-functions"></a>Funciones definidas por el usuario

**Las funciones definidas por el usuario** son subconsultas reutilizables que se pueden definir como parte de la propia consulta (**funciones ad hoc**) o conservarse como parte de los metadatos de la base de datos (**funciones almacenadas**). Las funciones definidas por el usuario se invocan mediante un **nombre**, se proporcionan cero o más **argumentos de entrada** (que pueden ser escalares o tabulares) y generan un valor único (que puede ser escalar o tabular) en función del **cuerpo**de la función.

Una función definida por el usuario pertenece a una de estas dos categorías:

* Funciones escalares 
* Funciones tabulares 

Los argumentos de entrada y la salida de la función determinan si es escalar o tabular, que establece cómo se podría usar. 

## <a name="scalar-function"></a>Función escalar

* Tiene cero argumentos de entrada o todos sus argumentos de entrada son valores escalares
* Genera un único valor escalar.
* Se puede usar siempre que se permita una expresión escalar.
* Solo puede utilizar el contexto de fila en el que está definido
* Solo puede hacer referencia a tablas (y vistas) que están en el esquema accesible

## <a name="tabular-function"></a>Función tabular

* Acepta uno o más argumentos de entrada tabulares y cero o más argumentos de entrada escalares, y/o:
* Genera un único valor tabular.

## <a name="function-names"></a>Nombres de función

Los nombres de función definidos por el usuario válidos deben seguir las mismas [reglas de nomenclatura de identificadores](../schema-entities/entity-names.md#identifier-naming-rules) que otras entidades.

El nombre también debe ser único en su ámbito de definición.

> [!NOTE]
> No se admite la sobrecarga de funciones. No se pueden definir varias funciones con el mismo nombre.

## <a name="input-arguments"></a>Argumentos de entrada

Las funciones definidas por el usuario válidas siguen estas reglas:

* Una función definida por el usuario tiene una lista fuertemente tipada de cero o más argumentos de entrada.
* Un argumento de entrada tiene un nombre, un tipo y (para los argumentos escalares) un [valor predeterminado](#default-values).
* El nombre de un argumento de entrada es un identificador.
* El tipo de un argumento de entrada es cualquiera de los tipos de datos escalares o un esquema tabular.

Sintácticamente, la lista de argumentos de entrada es una lista separada por comas de definiciones de argumentos, que se incluyen entre paréntesis. Cada definición de argumento se especifica como

```
ArgName:ArgType [= ArgDefaultValue]
```
 En el caso de los argumentos tabulares, *ArgType* tiene la misma sintaxis que la definición de tabla (paréntesis y una lista de pares de nombre y tipo de columna), con la compatibilidad adicional de solitarios `(*)` que indica "cualquier esquema tabular".

Por ejemplo:

|Sintaxis                        |Descripción de la lista de argumentos de entrada                                 |
|------------------------------|-----------------------------------------------------------------|
|`()`                          |Sin argumentos|
|`(s:string)`                  |Argumento escalar único denominado `s` tomar un valor de tipo`string`|
|`(a:long, b:bool=true)`       |Dos argumentos escalares, el segundo de los cuales tiene un valor predeterminado    |
|`(T1:(*), T2(r:real), b:bool)`|Tres argumentos (dos argumentos tabulares y un argumento escalar)  |

> [!NOTE]
> Cuando se utilizan argumentos de entrada tabulares y argumentos de entrada escalares, se colocan todos los argumentos de entrada tabulares delante de los argumentos de entrada escalares.

## <a name="examples"></a>Ejemplos

Una función escalar:

```kusto
let Add7 = (arg0:long = 5) { arg0 + 7 };
range x from 1 to 10 step 1
| extend x_plus_7 = Add7(x), five_plus_seven = Add7()
```

Función tabular que no toma ningún argumento:

```kusto
let tenNumbers = () { range x from 1 to 10 step 1};
tenNumbers
| extend x_plus_7 = x + 7
```

Función tabular que toma una entrada tabular y una entrada escalar:

```kusto
let MyFilter = (T:(x:long), v:long) {
  T | where x >= v
};
MyFilter((range x from 1 to 10 step 1), 9)
```

|x|
|---|
|9|
|10|

Función tabular que usa una entrada tabular sin ninguna columna especificada.
Cualquier tabla se puede pasar a una función y no se puede hacer referencia a ninguna columna de tabla dentro de la función.

```kusto
let MyDistinct = (T:(*)) {
  T | distinct *
};
MyDistinct((range x from 1 to 3 step 1))
```

|x|
|---|
|1|
|2|
|3|

## <a name="declaring-user-defined-functions"></a>Declarar funciones definidas por el usuario

La declaración de una función definida por el usuario proporciona:

* **Nombre** de función
* **Esquema** de función (parámetros aceptados, si los hay)
* **Cuerpo** de la función

> [!Note]
> No se admite la sobrecarga de funciones. No se pueden crear varias funciones con el mismo nombre y con distintos esquemas de entrada.

> [!TIP]
> Las funciones lambda no tienen un nombre y están enlazadas a un nombre mediante una [instrucción Let](../letstatement.md). Por lo tanto, se pueden considerar como funciones almacenadas definidas por el usuario.
> Ejemplo: Declaración de una función lambda que acepta dos argumentos ( `string` denominados `s` y `long` denominados `i` ). Devuelve el producto de la primera (después de convertirla en un número) y el segundo. La expresión lambda está enlazada al nombre `f` :

```kusto
let f=(s:string, i:long) {
    tolong(s) * i
};
```

El **cuerpo** de la función incluye:

* Exactamente una expresión, que proporciona el valor devuelto de la función (valor escalar o tabular).
* Cualquier número (cero o más) de [instrucciones Let](../letstatement.md), cuyo ámbito es el del cuerpo de la función. Si se especifica, las instrucciones Let deben preceder a la expresión que define el valor devuelto de la función.
* Cualquier número (cero o más) de [instrucciones de parámetros de consulta](../queryparametersstatement.md), que declaran parámetros de consulta usados por la función. Si se especifica, deben preceder a la expresión que define el valor devuelto de la función.

> [!NOTE]
> Otros tipos de [instrucciones de consulta](../statements.md) que se admiten en la consulta "nivel superior" no se admiten dentro del cuerpo de una función.

### <a name="examples-of-user-defined-functions"></a>Ejemplos de funciones definidas por el usuario 

**Función definida por el usuario que usa una instrucción Let**

En el ejemplo siguiente se enlaza el nombre `Test` a una función definida por el usuario (lambda) que hace uso de tres instrucciones Let. El resultado es `70` :

```kusto
let Test1 = (id: int) {
  let Test2 = 10;
  let Test3 = 10 + Test2 + id;
  let Test4 = (arg: int) {
      let Test5 = 20;
      Test2 + Test3 + Test5 + arg
  };
  Test4(10)
};
range x from 1 to Test1(10) step 1
| count
```

**Función definida por el usuario que define un valor predeterminado para un parámetro**

En el ejemplo siguiente se muestra una función que acepta tres argumentos. Los dos últimos tienen un valor predeterminado y no tienen que estar presentes en el sitio de llamada.

```kusto
let f = (a:long, b:string = "b.default", c:long = 0) {
  strcat(a, "-", b, "-", c)
};
print f(12, c=7) // Returns "12-b.default-7"
```

## <a name="invoking-a-user-defined-function"></a>Invocar una función definida por el usuario

Una función definida por el usuario que no toma ningún argumento se puede invocar por su nombre o por su nombre y una lista de argumentos vacía entre paréntesis.

Ejemplos:

```kusto
// Bind the identifier a to a user-defined function (lambda) that takes
// no arguments and returns a constant of type long:
let a=(){123};
// Invoke the function in two equivalent ways:
range x from 1 to 10 step 1
| extend y = x * a, z = x * a() 
```

```kusto
// Bind the identifier T to a user-defined function (lambda) that takes
// no arguments and returns a random two-by-two table:
let T=(){
  range x from 1 to 2 step 1
  | project x1 = rand(), x2 = rand()
};
// Invoke the function in two equivalent ways:
// (Note that the second invocation must be itself wrapped in
// an additional set of parentheses, as the union operator
// differentiates between "plain" names and expressions)
union T, (T())
```

Una función definida por el usuario que toma uno o más argumentos escalares se puede invocar utilizando el nombre de tabla y una lista de argumentos concreta entre paréntesis:

```kusto
let f=(a:string, b:string) {
  strcat(a, " (la la la)", b)
};
print f("hello", "world")
```

Una función definida por el usuario que toma uno o más argumentos de tabla (y cualquier número de argumentos escalares) se puede invocar utilizando el nombre de tabla y una lista de argumentos concretos entre paréntesis:

```kusto
let MyFilter = (T:(x:long), v:long) {
  T | where x >= v 
};
MyFilter((range x from 1 to 10 step 1), 9)
```

También puede utilizar el operador `invoke` para invocar una función definida por el usuario que toma uno o más argumentos de tabla y devuelve una tabla. Esta función es útil cuando el primer argumento de tabla concreto de la función es el origen del `invoke` operador:

```kusto
let append_to_column_a=(T:(a:string), what:string) {
    T | extend a=strcat(a, " ", what)
};
datatable (a:string) ["sad", "really", "sad"]
| invoke append_to_column_a(":-)")
```

## <a name="default-values"></a>Valores predeterminados

Las funciones pueden proporcionar valores predeterminados a algunos de sus parámetros en las siguientes condiciones:

* Solo se pueden proporcionar valores predeterminados para los parámetros escalares.
* Los valores predeterminados son siempre literales (constantes). No pueden ser cálculos arbitrarios.
* Los parámetros sin valores predeterminados siempre preceden a los parámetros que tienen un valor predeterminado.
* Los llamadores deben proporcionar el valor de todos los parámetros sin valores predeterminados organizados en el mismo orden que la declaración de función.
* Los llamadores no necesitan proporcionar el valor de los parámetros con valores predeterminados, pero pueden hacerlo.
* Los llamadores pueden proporcionar argumentos en un orden que no coincida con el orden de los parámetros. Si es así, deben asignar un nombre a sus argumentos.

En el ejemplo siguiente se devuelve una tabla con dos registros idénticos. En la primera invocación de `f` , los argumentos están completamente "codificados", por lo que cada uno de ellos recibe explícitamente un nombre:

```kusto
let f = (a:long, b:string = "b.default", c:long = 0) {
  strcat(a, "-", b, "-", c)
};
union
  (print x=f(c=7, a=12)), // "12-b.default-7"
  (print x=f(12, c=7))    // "12-b.default-7"
```

|x|
|---|
|12-b. valor predeterminado: 7|
|12-b. valor predeterminado: 7|

## <a name="view-functions"></a>Funciones de vista

Una función definida por el usuario que no toma ningún argumento y devuelve una expresión tabular se puede marcar como una **vista**. Marcar una función definida por el usuario como una vista significa que la función se comporta como una tabla cuando se realiza la resolución de nombres de tabla de caracteres comodín.
En el ejemplo siguiente se muestran dos funciones definidas por `T_view` el usuario, y `T_notview` , y se muestra cómo solo la primera de ellas se resuelve mediante la referencia de carácter comodín en la `union` :

```kusto
let T_view = view () { print x=1 };
let T_notview = () { print x=2 };
union T*
```

## <a name="restrictions"></a>Restricciones

Se aplican las restricciones que se indican a continuación:

* Las funciones definidas por el usuario no pueden pasar a la información de invocación de [toscalar ()](../toscalarfunction.md) que depende del contexto de fila en el que se llama a la función.
* Funciones definidas por el usuario que devuelven una expresión tabular can'tbe invocada con un argumento que varía con el contexto de la fila.
* No se puede invocar una función que contenga al menos una entrada tabular en un clúster remoto.
* No se puede invocar una función escalar en un clúster remoto.

El único lugar en el que se puede invocar una función definida por el usuario con un argumento que varía con el contexto de la fila es cuando la función definida por el usuario se compone solo de funciones escalares y no usa `toscalar()` .

**Ejemplo de restricción 1**

```kusto
// Supported:
// f is a scalar function that doesn't reference any tabular expression
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { now() + hours*1h };
Table2 | where Column != 123 | project d = f(10)

// Supported:
// f is a scalar function that references the tabular expression Table1,
// but is invoked with no reference to the current row context f(10):
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { toscalar(Table1 | summarize min(xdate) - hours*1h) };
Table2 | where Column != 123 | project d = f(10)

// Not supported:
// f is a scalar function that references the tabular expression Table1,
// and is invoked with a reference to the current row context f(Column):
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { toscalar(Table1 | summarize min(xdate) - hours*1h) };
Table2 | where Column != 123 | project d = f(Column)
```

**Ejemplo de restricción 2**

```kusto
// Not supported:
// f is a tabular function that is invoked in a context
// that expects a scalar value.
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { range x from 1 to hours step 1 | summarize make_list(x) };
Table2 | where Column != 123 | project d = f(Column)
```

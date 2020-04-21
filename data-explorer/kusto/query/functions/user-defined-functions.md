---
title: 'Funciones definidas por el usuario: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describen las funciones definidas por el usuario en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: e8372be303b1540e1421f226ed0fd0fe74d44545
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610241"
---
# <a name="user-defined-functions"></a>Funciones definidas por el usuario

Kusto admite funciones definidas por el usuario, que son:
* Parte de la propia consulta **(funciones ad hoc)** 
* Almacenado de forma persistente como parte de los metadatos de la base de datos (**funciones almacenadas**)

Una función definida por el usuario tiene:
* Un nombre:
    * Debe seguir las reglas de nomenclatura del [identificador](../schema-entities/entity-names.md#identifier-naming-rules)
    * Debe ser único en el ámbito de la definición con una especificación de tipo
* Una lista fuertemente tipada de parámetros de entrada:
    * Pueden ser expresiones escalares o tabulares
    * Los parámetros escalares se pueden proporcionar con un valor predeterminado, utilizado implícitamente cuando el llamador de la función no proporciona un valor para el parámetro (consulte [Valores predeterminados](#default-values), a continuación)
* Un valor devuelto fuertemente tipado, que puede ser escalar o tabular

Las entradas y la salida de una función determinan cómo y dónde se puede utilizar:

* **Una función escalar:** 
    * Es una función sin entradas, o con entradas escalares solamente, y que produce una salida escalar
    * Se puede utilizar siempre que se permita una expresión escalar
    * Sólo puede utilizar el contexto de fila en el que se define
    * Solo puede hacer referencia a tablas (y vistas) que se encuentran en el esquema accesible

Ejemplo:

```kusto
let Add7 = (arg0:long = 5) { arg0 + 7 };
range x from 1 to 10 step 1
| extend x_plus_7 = Add7(x), five_plus_seven = Add7()
```

* **Una función tabular:** 
    * Es una función sin entradas, o al menos una entrada tabular, y produce una salida tabular
    * Se puede utilizar siempre que se permita una expresión tabular 

> [!NOTE]
> Todos los parámetros tabulares deben aparecer antes que los parámetros escalares.

Ejemplo de una función tabular que no toma ningún argumento:

```kusto
let tenNumbers = () { range x from 1 to 10 step 1};
tenNumbers
| extend x_plus_7 = x + 7
```

Ejemplo de una función tabular que utiliza una entrada tabular y una entrada escalar:

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

Ejemplo de una función tabular que utiliza una entrada tabular sin ninguna columna especificada. Cualquier tabla se puede pasar a una función y no se puede hacer referencia a ninguna columna de tabla dentro de la función.

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

## <a name="declaring-user-defined-functions"></a>Declaración de funciones definidas por el usuario

La declaración de una función definida por el usuario proporciona:

* Nombre de **la** función
* Esquema de **función** (parámetros que acepta, si los hay)
* Cuerpo **de función**

> [!Note]
> No se admiten funciones de sobrecarga. Por ejemplo, no puede crear varias funciones con el mismo nombre y diferentes esquemas de entrada.

> [!TIP]
> Las funciones de Lambda no tienen un nombre y están enlazadas a un nombre mediante una [instrucción let.](../letstatement.md) Por lo tanto, pueden considerarse como funciones almacenadas definidas por el usuario.
> Ejemplo: Declaración para una función lambda `string` que `s` acepta `long` `i`dos argumentos (un llamado y un llamado ). Devuelve el producto del primero (después de convertirlo en un número) y el segundo. La lambda está enlazada al nombre: `f`

```kusto
let f=(s:string, i:long) {
    tolong(s) * i
};
```

El **cuerpo** de la función incluye:

* Exactamente una expresión, que proporciona el valor devuelto de la función (valor escalar o tabular).
* Cualquier número (cero o más) de [let instrucciones](../letstatement.md), cuyo ámbito es el del cuerpo de la función. Si se especifica, las instrucciones let deben preceder a la expresión que define el valor devuelto de la función.
* Cualquier número (cero o más) de instrucciones de parámetros de consulta, que declaran los [parámetros](../queryparametersstatement.md)de consulta utilizados por la función. Si se especifica, deben preceder a la expresión que define el valor devuelto de la función.

> [!NOTE]
> Otros tipos de instrucciones de [consulta](../statements.md) que se admiten en la consulta "nivel superior", no se admiten dentro de un cuerpo de función.

### <a name="examples-of-user-defined-functions"></a>Ejemplos de funciones definidas por el usuario 

**Función definida por el usuario que utiliza una instrucción let**

En el ejemplo siguiente `Test` se enlaza el nombre a una función definida por el usuario (lambda) que hace uso de tres instrucciones let. La salida `70`es:

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

Una función definida por el usuario que no toma ningún argumento se puede invocar por su nombre o por su nombre con una lista de argumentos vacía entre paréntesis.

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

Una función definida por el usuario que toma uno o más argumentos escalares se puede invocar mediante el nombre de tabla y una lista de argumentos concretos entre paréntesis:

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

También puede utilizar `invoke` el operador para invocar una función definida por el usuario que toma uno o varios argumentos de tabla y devuelve una tabla. Esto es útil cuando el primer argumento de tabla `invoke` concreto para la función es el origen del operador:

```kusto
let append_to_column_a=(T:(a:string), what:string) {
    T | extend a=strcat(a, " ", what)
};
datatable (a:string) ["sad", "really", "sad"]
| invoke append_to_column_a(":-)")
```

## <a name="default-values"></a>Valores predeterminados

Las funciones pueden proporcionar valores predeterminados a algunos de sus parámetros en las siguientes condiciones:

* Los valores predeterminados solo se pueden proporcionar para los parámetros escalares.
* Los valores predeterminados siempre son literales (constantes). No pueden ser cálculos arbitrarios.
* Los parámetros sin valor predeterminado siempre preceden a los parámetros que tienen un valor predeterminado.
* Los llamadores deben proporcionar el valor de todos los parámetros sin valores predeterminados dispuestos en el mismo orden que la declaración de función.
* Los llamadores no necesitan proporcionar el valor de los parámetros con valores predeterminados, pero pueden hacerlo.
* Los llamadores pueden proporcionar argumentos en un orden que no coincida con el orden de los parámetros. Si es así, deben nombrar sus argumentos.

En el ejemplo siguiente se devuelve una tabla con dos registros idénticos. En la primera `f`invocación de , los argumentos están completamente "codificados", por lo que a cada uno se le da explícitamente un nombre:

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
|12-b.default-7|
|12-b.default-7|

## <a name="view-functions"></a>Ver funciones

Una función definida por el usuario que no toma argumentos y devuelve una expresión tabular se puede marcar como **vista.** Marcar una función definida por el usuario como una vista significa que la función se comporta como una tabla cada vez que se realiza la resolución de nombres de tabla comodín.
En el ejemplo siguiente se `T_view` muestran dos funciones definidas por el usuario `T_notview`y `union`, y se muestra cómo solo se resuelve la primera mediante la referencia comodín en :

```kusto
let T_view = view () { print x=1 };
let T_notview = () { print x=2 };
union T*
```

## <a name="user-defined-functions-usage-restrictions"></a>Restricciones de uso de funciones definidas por el usuario

Se aplican las restricciones que se indican a continuación:

* Las funciones definidas por el usuario no pueden pasar a [toscalar()](../toscalarfunction.md) información de invocación que depende del contexto de fila en el que se llama a la función.
* Las funciones definidas por el usuario que devuelven una expresión tabular no se pueden invocar con un argumento que varíe con el contexto de fila.
* Una función que toma al menos una entrada tabular no se puede invocar en un clúster remoto.
* No se puede invocar una función escalar en un clúster remoto.

El único lugar en el que se puede invocar una función definida por el usuario con un argumento que varía `toscalar()`con el contexto de fila es cuando la función definida por el usuario se compone solo de funciones escalares y no usa .

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

**Ejemplo de Restricción 2**

```kusto
// Not supported:
// f is a tabular function that is invoked in a context
// that expects a scalar value.
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { range x from 1 to hours step 1 | summarize make_list(x) };
Table2 | where Column != 123 | project d = f(Column)
```

---
title: operador mv-apply - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el operador mv-apply en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f24bf7721707aa1ba3ae9f0aad49b247f08c2498
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512315"
---
# <a name="mv-apply-operator"></a>Operador mv-apply

El operador mv-apply expande cada registro de su tabla de entrada en una subtabla, aplica una subconsulta a cada subtabla y devuelve la unión de los resultados de todas las subconsultas.

Por ejemplo, supongamos `T` que `Metric` una `dynamic` tabla tiene una `real` columna de tipo cuyos valores son matrices de números. La siguiente consulta buscará los `Metric` dos valores más grandes en cada valor y devolverá los registros correspondientes a estos valores.

```kusto
T | mv-apply Metric to typeof(real) on (top 2 by Metric desc)
```

En general, se puede considerar que el operador mv-apply tiene los siguientes pasos de procesamiento:

1. Utiliza el operador [mv-expand](./mvexpandoperator.md) para expandir cada registro de la entrada en subtablas.
2. Aplica la subconsulta para cada una de las subtablas.
3. Antepone cero o más columnas a cada subtabla resultante, que contiene los valores (repetidos si es necesario) de las columnas de origen que no se están expandiendo.
4. Devuelve la unión de los resultados.

El operador mv-expand obtiene las siguientes entradas:

1. Una o más expresiones que se evalúan en matrices dinámicas para expandir.
   El número de registros en cada subtabla expandida es la longitud máxima de cada una de esas matrices dinámicas. (Si se especifican varias expresiones, pero las matrices correspondientes son de diferentes longitudes, se introducen valores nulos si es necesario.)

2. Opcionalmente, los nombres para asignar los valores de las expresiones, después de la expansión.
   Estos se convierten en los nombres de las columnas de las subtablas.
   Si no se especifica, se utiliza el nombre original de la columna (si la expresión es una referencia de columna) o se utiliza un nombre aleatorio (de lo contrario).

   > [!NOTE]
   > Se recomienda utilizar los nombres de columna predeterminados.

3. Los tipos de datos de los elementos de esas matrices dinámicas, después de la expansión.
   Estos se convierten en los tipos de columna de las columnas de las subtablas.
   Si no se especifica, se utiliza `dynamic`.

4. Opcionalmente, el nombre de una columna que se va a agregar a las subtablas que especifica el índice basado en 0 del elemento de la matriz que dio lugar al registro de la subtabla.

5. Opcionalmente, el número máximo de elementos de matriz que se expandirán.

El operador mv-apply se puede considerar como una generalización del operador [mv-expand](./mvexpandoperator.md) (de hecho, el segundo puede ser implementado por el primero, si la subconsulta incluye sólo proyecciones.)

**Sintaxis**

*T* `|` T `mv-apply` [*ItemIndex*] *ColumnsToExpand* [*RowLimit*] `on` `(` *SubQuery*`)`

Donde *ItemIndex* tiene la sintaxis:

`with_itemindex``=` *IndexColumnName*

*ColumnsToExpand* es una lista separada por comas de uno o más elementos del formulario:

[*Nombre* `=`] *ArrayExpression* `to` `typeof` `(`[ *Typename*`)`]

*RowLimit* es simplemente:

`limit`*RowLimit*

y *SubQuery* tiene la misma sintaxis de cualquier instrucción de consulta.

**Argumentos**

* *ItemIndex*: Si se utiliza, indica el `long` nombre de una columna de tipo que se anexa a la entrada como parte de la fase de expansión de matriz e indica el índice de matriz basado en 0 del valor expandido.

* *Nombre*: Si se utiliza, el nombre para asignar los valores expandidos de matriz de cada expresión expandida de matriz.
  (Si no se especifica, el nombre de la columna se utilizará si está disponible, o un nombre aleatorio generado si *ArrayExpression* no es un nombre de columna simple.)

* *ArrayExpression*: Una `dynamic` expresión de tipo cuyos valores se expandirán en matriz.
  Si la expresión es el nombre de una columna de la entrada, la columna de entrada se quita de la entrada y aparecerá una nueva columna con el mismo nombre (o *ColumnName* si se especifica) en la salida.

* *Typename*: Si se utiliza, el nombre del `dynamic` tipo que toma la matriz *ArrayExpression.* Los elementos que no se ajustan a este tipo se reemplazarán por un valor nulo.
  (Si no se `dynamic` especifica, se utiliza de forma predeterminada.)

* *RowLimit*: Si se utiliza, un límite en el número de registros que se generarán a partir de cada registro de la entrada.
  (Si no se especifica, se utiliza 2147483647.)

* *SubQuery*: Una expresión de consulta tabular con un origen tabular implícito que se aplica a cada subtabla expandida por matriz.

**Notas**

* A diferencia del operador [mv-expand,](./mvexpandoperator.md) el operador mv-apply solo admite la expansión de matriz. No hay soporte para ampliar las bolsas de propiedades.

**Ejemplos**

## <a name="getting-the-largest-element-from-the-array"></a>Obtener el elemento más grande de la matriz

```kusto
let _data =
range x from 1 to 8 step 1
| summarize l=make_list(x) by xMod2 = x % 2;
_data
| mv-apply element=l to typeof(long) on 
(
   top 1 by element
)
```

|xMod2|l           |elemento|
|-----|------------|-------|
|1    |[1, 3, 5, 7]|7      |
|0    |[2, 4, 6, 8]|8      |

## <a name="calculating-sum-of-largest-two-elments-in-an-array"></a>Cálculo de la suma de dos elments más grandes en una matriz

```kusto
let _data =
range x from 1 to 8 step 1
| summarize l=make_list(x) by xMod2 = x % 2;
_data
| mv-apply l to typeof(long) on
(
   top 2 by l
   | summarize SumOfTop2=sum(l)
)
```

|xMod2|l        |SumOfTop2|
|-----|---------|---------|
|1    |[1,3,5,7]|12       |
|0    |[2,4,6,8]|14       |


## <a name="using-with_itemindex-for-working-with-subset-of-the-array"></a>Uso `with_itemindex` para trabajar con el subconjunto de la matriz

```kusto
let _data =
range x from 1 to 10 step 1
| summarize l=make_list(x) by xMod2 = x % 2;
_data
| mv-apply with_itemindex=index element=l to typeof(long) on 
(
   // here you have 'index' column
   where index >= 3
)
| project index, element
```

|índice|elemento|
|---|---|
|3|7|
|4|9|
|3|8|
|4|10|

## <a name="using-mv-apply-operator-to-sort-the-output-of-makelist-aggregate-by-some-key"></a>Uso `mv-apply` del operador para `makelist` ordenar la salida del agregado por alguna clave

```kusto
datatable(command:string, command_time:datetime, user_id:string)
[
    'chmod',        datetime(2019-07-15),   "user1",
    'ls',           datetime(2019-07-02),   "user1",
    'dir',          datetime(2019-07-22),   "user1",
    'mkdir',        datetime(2019-07-14),   "user1",
    'rm',           datetime(2019-07-27),   "user1",
    'pwd',          datetime(2019-07-25),   "user1",
    'rm',           datetime(2019-07-23),   "user2",
    'pwd',          datetime(2019-07-25),   "user2",
]
| summarize commands_details = make_list(pack('command', command, 'command_time', command_time)) by user_id
| mv-apply command_details = commands_details on
(
    order by todatetime(command_details['command_time']) asc
    | summarize make_list(tostring(command_details['command']))
)
| project-away commands_details 
```

|user_id|list_command_details_command|
|---|---|
|user1|[<br>  "ls",<br>  "mkdir",<br>  "chmod",<br>  "dir",<br>  "pwd",<br>  "Rm"<br>]|
|user2|[<br>  "rm",<br>  "Buena"<br>]|


**Vea también**

* [mv-expand](./mvexpandoperator.md) operador.
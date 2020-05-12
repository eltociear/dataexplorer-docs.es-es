---
title: 'operador MV-Apply: Azure Explorador de datos'
description: En este artículo se describe el operador MV-Apply en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bb0ab7fd0f3508388a29d4931cea770c8619e083
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226438"
---
# <a name="mv-apply-operator"></a>Operador mv-apply

El `mv-apply` operador expande cada registro de la tabla de entrada en una subtabla, aplica una subconsulta a cada subtabla y devuelve la Unión de los resultados de todas las subconsultas.

Por ejemplo, suponga que una tabla `T` tiene una columna `Metric` de tipo `dynamic` cuyos valores son matrices de `real` números. La consulta siguiente buscará los dos valores más grandes de cada `Metric` valor y devolverá los registros correspondientes a estos valores.

```kusto
T | mv-apply Metric to typeof(real) on (top 2 by Metric desc)
```

El `mv-apply` operador tiene los siguientes pasos de procesamiento:

1. Utiliza el [`mv-expand`](./mvexpandoperator.md) operador para expandir cada registro de la entrada en subtablas.
1. Aplica la subconsulta para cada una de las subtablas.
1. Agrega cero o más columnas a la subtabla resultante. Estas columnas contienen los valores de las columnas de origen que no se expanden y se repiten cuando sea necesario.
1. Devuelve la Unión de los resultados.

El `mv-expand` operador obtiene las siguientes entradas:

1. Una o varias expresiones que se evalúan como matrices dinámicas que se van a expandir.
   El número de registros de cada subtabla expandida es la longitud máxima de cada una de esas matrices dinámicas. Se agregan valores NULL cuando se especifican varias expresiones y las matrices correspondientes tienen longitudes diferentes.

1. Opcionalmente, los nombres que se asignan a los valores de las expresiones después de la expansión.
   Estos nombres se convierten en los nombres de las columnas de las subtablas.
   Si no se especifica, se utiliza el nombre original de la columna cuando la expresión es una referencia de columna. En caso contrario, se utiliza un nombre aleatorio. 

   > [!NOTE]
   > Se recomienda usar los nombres de columna predeterminados.

1. Los tipos de datos de los elementos de esas matrices dinámicas, después de la expansión.
   Estos se convierten en los tipos de columna de las columnas de las subtablas.
   Si no se especifica, se utiliza `dynamic`.

1. Opcionalmente, el nombre de una columna que se va a agregar a las subtablas que especifica el índice de base 0 del elemento de la matriz que dio como resultado el registro de la subtabla.

1. Opcionalmente, el número máximo de elementos de matriz que se van a expandir.

El `mv-apply` operador se puede considerar como una generalización del [`mv-expand`](./mvexpandoperator.md) operador (de hecho, el último puede ser implementado por el primero, si la subconsulta incluye solo proyecciones).

**Sintaxis**

*T* `|` `mv-apply` [*itemIndex*] *ColumnsToExpand* [*ROWLIMIT*] `on` `(` *subconsulta*`)`

Donde *itemIndex* tiene la sintaxis:

`with_itemindex``=` *IndexColumnName*

*ColumnsToExpand* es una lista separada por comas de uno o más elementos del formulario:

[*Nombre* `=` ] *ArrayExpression* [ `to` `typeof` `(` *TypeName* `)` ]

*ROWLIMIT* es simplemente:

`limit`*ROWLIMIT*

y la *subconsulta* tienen la misma sintaxis que cualquier instrucción de consulta.

**Argumentos**

* *ItemIndex*: si se usa, indica el nombre de una columna de tipo `long` que se anexa a la entrada como parte de la fase de expansión de la matriz e indica el índice de matriz basado en 0 del valor expandido.

* *Name*: si se usa, el nombre para asignar los valores de la matriz expandida de cada expresión expandida por la matriz.
  Si no se especifica, se utilizará el nombre de la columna si está disponible.
  Si *ArrayExpression* no es un nombre de columna simple, se genera un nombre aleatorio.

* *ArrayExpression*: una expresión de tipo `dynamic` cuyos valores se expandirán a la matriz.
  Si la expresión es el nombre de una columna de la entrada, la columna de entrada se quita de la entrada y aparece una nueva columna con el mismo nombre (o *columnName* si se especifica) en la salida.

* *TypeName*: si se usa, es el nombre del tipo que toman los elementos individuales de la `dynamic` matriz *ArrayExpression* . Los elementos que no se ajustan a este tipo se reemplazarán por un valor null.
  (Si `dynamic` no se especifica, se usa de forma predeterminada).

* *ROWLIMIT*: si se usa, límite en el número de registros que se generarán a partir de cada registro de la entrada.
  (Si no se especifica, se usa 2147483647).

* *Subconsulta*: expresión de consulta tabular con un origen tabular implícito que se aplica a cada subtabla expandida por la matriz.

**Notas**

* A diferencia del [`mv-expand`](./mvexpandoperator.md) operador, el `mv-apply` operador solo admite la expansión de la matriz. No se admite la expansión de bolsas de propiedades.

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

|`xMod2`|l           |elemento|
|-----|------------|-------|
|1    |[1, 3, 5, 7]|7      |
|0    |[2, 4, 6, 8]|8      |

## <a name="calculating-the-sum-of-the-largest-two-elements-in-an-array"></a>Calcular la suma de los dos elementos más grandes de una matriz

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

|`xMod2`|l        |SumOfTop2|
|-----|---------|---------|
|1    |[1, 3, 5, 7]|12       |
|0    |[2, 4, 6, 8]|14       |


## <a name="using-with_itemindex-for-working-with-a-subset-of-the-array"></a>Usar `with_itemindex` para trabajar con un subconjunto de la matriz

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

## <a name="using-the-mv-apply-operator-to-sort-the-output-of-makelist-aggregate-by-some-key"></a>Usar el `mv-apply` operador para ordenar la salida de `makelist` Aggregate mediante alguna clave

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

|`user_id`|`list_command_details_command`|
|---|---|
|user1|[<br>  "LS",<br>  "mkdir",<br>  "chmod",<br>  "dir",<br>  "pwd",<br>  Dist<br>]|
|user2|[<br>  "RM",<br>  pwd<br>]|


**Vea también**

* operador [MV-Expand](./mvexpandoperator.md) .

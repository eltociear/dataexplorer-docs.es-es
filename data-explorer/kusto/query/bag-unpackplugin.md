---
title: 'complemento de bag_unpack: Azure Explorador de datos'
description: En este artículo se describe bag_unpack complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 06/15/2020
ms.openlocfilehash: 1823a9d875c6294f360fbce77fcb5e1d2c968019
ms.sourcegitcommit: 3848b8db4c3a16bda91c4a5b7b8b2e1088458a3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84818555"
---
# <a name="bag_unpack-plugin"></a>complemento de bag_unpack

El `bag_unpack` complemento desempaqueta una sola columna de tipo `dynamic` tratando cada ranura de nivel superior de la bolsa de propiedades como una columna.

    T | evaluate bag_unpack(col1)

**Sintaxis**

*T* ( `|` `evaluate` `bag_unpack(` *columna* ) [ `,` *OutputColumnPrefix* ] [ `,` *columnsConlict* ] [ `,` *ignoredProperties* ]`)`

**Argumentos**

* *T*: la entrada tabular cuya *columna* de columna se va a desempaquetar.
* *Columna*: columna de *T* que se va a desempaquetar. Debe ser de tipo `dynamic`.
* *OutputColumnPrefix*: prefijo común que se va a agregar a todas las columnas generadas por el complemento. Este argumento es opcional.
* *columnsConlict*: una dirección para la resolución de conflictos de columna. Este argumento es opcional. Cuando se proporciona el argumento, se espera que sea un literal de cadena que coincida con uno de los valores siguientes:
    - `error`-la consulta genera un error (valor predeterminado)
    - `replace_source`-la columna de origen se ha reemplazado
    - `keep_source`-la columna de origen se mantiene
* *ignoredProperties*: conjunto opcional de propiedades de contenedor que se va a omitir. Cuando se proporciona el argumento, se espera que sea una constante de `dynamic` matriz con uno o más literales de cadena.

**Devuelve**

El `bag_unpack` complemento devuelve una tabla con tantos registros como entrada tabular (*T*). El esquema de la tabla es el mismo que el del esquema de su entrada tabular con las siguientes modificaciones:

* Se quita la columna de entrada especificada (*columna*).

* El esquema se extiende con tantas columnas como espacios distintivos en los valores de bolsa de propiedades de nivel superior de *T*. El nombre de cada columna corresponde al nombre de cada ranura, opcionalmente con el prefijo *OutputColumnPrefix*. Su tipo es el tipo de la ranura (si todos los valores de la misma ranura tienen el mismo tipo) o `dynamic` (si los valores difieren en el tipo).

**Notas**

El esquema de salida del complemento depende de los valores de los datos, lo que lo convierte en "imprevisible" como los propios datos. Como tal, varias ejecuciones del complemento con una entrada de datos diferente pueden generar un esquema de salida diferente.

Los datos de entrada para el complemento deben ser tales, por lo que el esquema de salida sigue todas las reglas de un esquema tabular. En concreto:

1. Un nombre de columna de salida no puede ser el mismo que el de una columna existente en la entrada tabular *T* a menos que sea la columna que se va a desempaquetar (*columna*), ya que se producirán dos columnas con el mismo nombre.

2. Todos los nombres de ranura, al prefijarse por *OutputColumnPrefix*, deben ser nombres de entidad válidos y seguir las [reglas de nomenclatura de identificadores](./schema-entities/entity-names.md#identifier-naming-rules).

**Ejemplos**

Expandir un contenedor:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20}),
    dynamic({"Name": "Dave", "Age":40}),
    dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d)
```

|Nombre  |Age|
|------|---|
|Juan  |20 |
|Dave  |40 |
|Jasmine|30 |

Expandir un contenedor y usar la `OutputColumnPrefix` opción para generar nombres de columna que empiecen por el prefijo ' Property_ ':

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20}),
    dynamic({"Name": "Dave", "Age":40}),
    dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d, 'Property_')
```

|Property_Name|Property_Age|
|---|---|
|Juan|20|
|Dave|40|
|Jasmine|30|

Expandir un contenedor y usar `columnsConlict` la opción para resolver conflictos entre las columnas existentes y las columnas generadas por el `bag_unpack()` operador.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(Name:string, d:dynamic)
[
    'Old_name', dynamic({"Name": "John", "Age":20}),
    'Old_name', dynamic({"Name": "Dave", "Age":40}),
    'Old_name', dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d, columnsConlict='replace_source') // Use new name
```

|Nombre|Age|
|---|---|
|Juan|20|
|Dave|40|
|Jasmine|30|

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(Name:string, d:dynamic)
[
    'Old_name', dynamic({"Name": "John", "Age":20}),
    'Old_name', dynamic({"Name": "Dave", "Age":40}),
    'Old_name', dynamic({"Name": "Jasmine", "Age":30}),
]
| evaluate bag_unpack(d, columnsConlict='keep_source') // Keep old name
```

|Nombre|Age|
|---|---|
|Old_name|20|
|Old_name|40|
|Old_name|30|

Expandir un contenedor y usar la `ignoredProperties` opción para omitir ciertas propiedades existentes en el contenedor de propiedades.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20, "Address": "Address-1" }),
    dynamic({"Name": "Dave", "Age":40, "Address": "Address-2"}),
    dynamic({"Name": "Jasmine", "Age":30, "Address": "Address-3"}),
]
// Ignore 'Age' and 'Address' properties
| evaluate bag_unpack(d, ignoredProperties=dynamic(['Address', 'Age']))
```

|Nombre|
|---|
|Juan|
|Dave|
|Jasmine|

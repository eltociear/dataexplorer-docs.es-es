---
title: bag_unpack complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe bag_unpack complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/21/2019
ms.openlocfilehash: 374ff3ca8101e24a53926a78a1b8781447f32dbc
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518231"
---
# <a name="bag_unpack-plugin"></a>bag_unpack plugin

El `bag_unpack` plugin desempaqueta una `dynamic` sola columna de tipo tratando cada ranura de nivel superior de la bolsa de propiedades como una columna.

    T | evaluate bag_unpack(col1)

**Sintaxis**

*T* `|` `evaluate` `bag_unpack(` *Column* Columna `,` T [ *OutputColumnPrefix* ]`)`

**Argumentos**

* *T*: La entrada tabular cuya columna *columna* se va a desempaquetar.
* *Columna*: La columna de *T* para desempaquetar. Debe ser de tipo `dynamic`.
* *OutputColumnPrefix*: Un prefijo común para agregar a todas las columnas producidas por el plugin.
  Opcional.

**Devuelve**

El `bag_unpack` plugin devuelve una tabla con tantos registros como su entrada tabular (*T*). El esquema de la tabla es el mismo que el de su entrada tabular con las siguientes modificaciones:

* Se elimina la columna de entrada especificada (*Columna*).

* El esquema se amplía con tantas columnas como ranuras distintas en los valores de contenedor de propiedades de nivel superior de *T*. El nombre de cada columna corresponde al nombre de cada ranura, prefijado opcionalmente por *OutputColumnPrefix*. Su tipo es el tipo de la ranura (si todos los `dynamic` valores de la misma ranura tienen el mismo tipo) o (si los valores difieren en tipo).

**Notas**

El esquema de salida del plugin depende de los valores de datos, por lo que es tan "impredecible" como los datos en sí. Por lo tanto, varias ejecuciones del plugin con diferentes entradas de datos pueden producir un esquema de salida diferente.

Los datos de entrada al complemento deben ser tales, que el esquema de salida cumpla con todas las reglas para un esquema tabular. En concreto:

1. Un nombre de columna de salida no puede ser el mismo que una columna existente en la entrada tabular *T* a menos que sea la columna que se va a desempaquetar (*Columna*) sí mismo, ya que esto producirá dos columnas con el mismo nombre.

2. Todos los nombres de slot, cuando están prefijados por *OutputColumnPrefix*, deben ser nombres de entidad válidos y cumplir con las reglas de nomenclatura de [identificadores.](./schema-entities/entity-names.md#identifier-naming-rules)

**Ejemplo**

```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20}),
    dynamic({"Name": "Dave", "Age":40}),
    dynamic({"Name": "Smitha", "Age":30}),
]
| evaluate bag_unpack(d)
```

|NOMBRE  |Age|
|------|---|
|John  |20 |
|Dave  |40 |
|Smitha|30 |
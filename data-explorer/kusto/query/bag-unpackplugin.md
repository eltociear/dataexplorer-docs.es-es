---
title: 'complemento de bag_unpack: Azure Explorador de datos'
description: En este artículo se describe bag_unpack complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/21/2019
ms.openlocfilehash: fd8b0968d90f1c5239cae80c3be9c2a32d0603d6
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225503"
---
# <a name="bag_unpack-plugin"></a>complemento de bag_unpack

El `bag_unpack` complemento desempaqueta una sola columna de tipo `dynamic` tratando cada ranura de nivel superior de la bolsa de propiedades como una columna.

    T | evaluate bag_unpack(col1)

**Sintaxis**

*T* `|` `evaluate` `bag_unpack(` ( *columna* `,` ) [ *OutputColumnPrefix* ]`)`

**Argumentos**

* *T*: la entrada tabular cuya *columna* de columna se va a desempaquetar.
* *Columna*: columna de *T* que se va a desempaquetar. Debe ser de tipo `dynamic`.
* *OutputColumnPrefix*: prefijo común que se va a agregar a todas las columnas generadas por el complemento.
  Opcional.

**Devuelve**

El `bag_unpack` complemento devuelve una tabla con tantos registros como entrada tabular (*T*). El esquema de la tabla es el mismo que el de la entrada tabular con las siguientes modificaciones:

* Se quita la columna de entrada especificada (*columna*).

* El esquema se extiende con tantas columnas como espacios distintivos en los valores de bolsa de propiedades de nivel superior de *T*. El nombre de cada columna corresponde al nombre de cada ranura, opcionalmente con el prefijo *OutputColumnPrefix*. Su tipo es el tipo de la ranura (si todos los valores de la misma ranura tienen el mismo tipo) o `dynamic` (si los valores difieren en el tipo).

**Notas**

El esquema de salida del complemento depende de los valores de los datos, lo que lo convierte en "imprevisible" como los propios datos. Por lo tanto, varias ejecuciones del complemento con una entrada de datos diferente pueden generar un esquema de salida diferente.

Los datos de entrada para el complemento deben ser tales, de modo que el esquema de salida cumpla todas las reglas de un esquema tabular. En concreto:

1. Un nombre de columna de salida no puede ser el mismo que el de una columna existente en la entrada tabular *T* a menos que sea la columna que se va a desempaquetar (*columna*), ya que se producirán dos columnas con el mismo nombre.

2. Todos los nombres de ranura, al prefijarse por *OutputColumnPrefix*, deben ser nombres de entidad válidos y cumplir las [reglas de nomenclatura de identificadores](./schema-entities/entity-names.md#identifier-naming-rules).

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
datatable(d:dynamic)
[
    dynamic({"Name": "John", "Age":20}),
    dynamic({"Name": "Dave", "Age":40}),
    dynamic({"Name": "Smitha", "Age":30}),
]
| evaluate bag_unpack(d)
```

|Nombre  |Age|
|------|---|
|Juan  |20 |
|Dave  |40 |
|Smitha|30 |

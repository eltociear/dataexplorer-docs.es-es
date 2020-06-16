---
title: 'Cursores de base de datos: Azure Explorador de datos'
description: En este artículo se describen los cursores de base de datos de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 75dc0aa0ff23bfb4f08be9fac84fa34cf9526508
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780633"
---
# <a name="database-cursors"></a>Cursores de base de datos

Un **cursor de base** de datos es un objeto de nivel de base de datos que permite consultar una base de datos varias veces. Obtendrá resultados coherentes incluso si hay `data-append` `data-retention` operaciones de o en paralelo con las consultas.

Los cursores de base de datos están diseñados para abordar dos escenarios importantes:

* La capacidad de repetir la misma consulta varias veces y obtener los mismos resultados, siempre y cuando la consulta indique "el mismo conjunto de datos".

* La capacidad de realizar una consulta "exactamente una vez". Esta consulta solo "ve" los datos que una consulta anterior no ha visto, porque los datos no estaban disponibles.
   La consulta le permite iterar, por ejemplo, a través de todos los datos recién llegados en una tabla sin temor a procesar el mismo registro dos veces u omitir los registros por equivocación.

El cursor de la base de datos se representa en el lenguaje de consulta como un valor escalar de tipo `string` . El valor real se debe considerar opaco y no se admite ninguna operación que no sea para guardar su valor o usar las funciones de cursor que se indican a continuación.

## <a name="cursor-functions"></a>Funciones de cursor

Kusto proporciona tres funciones para ayudar a implementar los dos escenarios anteriores:

* [cursor_current ()](../query/cursorcurrent.md): Utilice esta función para recuperar el valor actual del cursor de la base de datos.
   Puede usar este valor como argumento para las otras dos funciones.
   Esta función también tiene un sinónimo, `current_cursor()` .

* [cursor_after (RHS: cadena)](../query/cursorafterfunction.md): esta función especial se puede usar en registros de tabla que tienen habilitada la [Directiva IngestionTime](ingestiontime-policy.md) . Devuelve un valor escalar de tipo `bool` que indica si el valor del cursor de la base de datos del registro `ingestion_time()` viene después del valor del cursor de la `rhs` base de datos.

* [cursor_before_or_at (RHS: cadena)](../query/cursorbeforeoratfunction.md): esta función especial se puede usar en los registros de la tabla que tienen habilitada la [Directiva IngestionTime](ingestiontime-policy.md) . Devuelve un valor escalar de tipo `bool` que indica si el valor del cursor de la base de datos del registro `ingestion_time()` viene después del valor del cursor de la `rhs` base de datos.

Las dos funciones especiales ( `cursor_after` y `cursor_before_or_at` ) también tienen un efecto secundario: cuando se usan, Kusto emitirá el **valor actual del cursor de la base de datos** al `@ExtendedProperties` conjunto de resultados de la consulta. El nombre de la propiedad del cursor es `Cursor` , y su valor es un único `string` . 

Por ejemplo:

```json
{"Cursor" : "636040929866477946"}
```

## <a name="restrictions"></a>Restricciones

Los cursores de base de datos solo se pueden usar con tablas para las que se ha habilitado la [Directiva IngestionTime](ingestiontime-policy.md) . Cada registro de una tabla de este tipo se asocia con el valor del cursor de la base de datos que estaba en vigor cuando se ingesta el registro.
Como tal, se puede utilizar la función [ingestion_time ()](../query/ingestiontimefunction.md) .

El objeto cursor de base de datos no contiene ningún valor significativo a menos que la base de datos tenga al menos una tabla que tenga definida una [Directiva IngestionTime](ingestiontime-policy.md) .
Se garantiza que este valor se actualiza, tal y como lo necesita el historial de ingesta, en esas tablas y se ejecutan las consultas que hacen referencia a dichas tablas. Podría actualizarse o no en otros casos.

En primer lugar, el proceso de ingesta confirma los datos para que estén disponibles para realizar consultas y, a continuación, asigna un valor de cursor real a cada registro. Si intenta consultar los datos inmediatamente después de la finalización de la ingesta mediante un cursor de base de datos, es posible que los resultados no incorporen todavía los últimos registros agregados, ya que aún no se les ha asignado el valor del cursor. Además, recuperar repetidamente el valor actual del cursor de la base de datos puede devolver el mismo valor, incluso si la ingesta se realizó en entre, porque solo una confirmación del cursor puede actualizar su valor.

## <a name="example-processing-records-exactly-once"></a>Ejemplo: procesamiento de registros exactamente una vez

En el caso de una tabla `Employees` con `[Name, Salary]` el esquema, para procesar continuamente los nuevos registros a medida que se ingesta en la tabla, use el siguiente proceso:

```kusto
// [Once] Enable the IngestionTime policy on table Employees
.set table Employees policy ingestiontime true

// [Once] Get all the data that the Employees table currently holds 
Employees | where cursor_after('')

// The query above will return the database cursor value in
// the @ExtendedProperties result set. Lets assume that it returns
// the value '636040929866477946'

// [Many] Get all the data that was added to the Employees table
// since the previous query was run using the previously-returned
// database cursor 
Employees | where cursor_after('636040929866477946') // -> 636040929866477950

Employees | where cursor_after('636040929866477950') // -> 636040929866479999

Employees | where cursor_after('636040929866479999') // -> 636040939866479000
```

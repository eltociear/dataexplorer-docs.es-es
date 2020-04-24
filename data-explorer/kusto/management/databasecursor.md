---
title: 'Cursores de base de datos: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen los cursores de base de datos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 90ec677a7eaf1f326509828b5415b022742fd9ed
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521325"
---
# <a name="database-cursors"></a>Cursores de base de datos

Un cursor de **base** de datos es un objeto de nivel de base de datos que permite consultar una base de datos varias veces y obtener resultados coherentes incluso si hay operaciones de retención de datos/anexadores de datos en curso que se realizan en paralelo con las consultas.

Los cursores de base de datos están diseñados para abordar dos escenarios importantes:

* La capacidad de repetir la misma consulta varias veces y obtener los mismos resultados, siempre que la consulta indique el "mismo conjunto de datos".

* La capacidad de realizar una consulta "exactamente una vez" (una consulta que solo "ve" los datos que una consulta anterior no vio porque los datos no estaban disponibles entonces).
   Esto permite, por ejemplo, recorrer en iteración todos los datos recién llegados en una tabla sin temor a procesar el mismo registro dos veces o omitir registros por error.

El cursor de base de datos se representa en `string`el lenguaje de consulta como un valor escalar de tipo . El valor real debe considerarse opaco y no hay soporte para ninguna operación que no sea guardar su valor y/o usar las funciones del cursor que se indican a continuación.

## <a name="cursor-functions"></a>Funciones del cursor

Kusto proporciona tres funciones para ayudar a implementar los dos escenarios anteriores:

* [cursor_current()](../query/cursorcurrent.md): Utilice esta función para recuperar el valor actual del cursor de la base de datos.
   Puede utilizar este valor como argumento para las otras dos funciones.
   Esta función también `current_cursor()`tiene un sinónimo, .

* [cursor_after(rhs:string):](../query/cursorafterfunction.md)esta función especial se puede utilizar en registros de tabla que tienen habilitada la [directiva IngestionTime.](ingestiontime-policy.md) Devuelve un valor escalar de `bool` tipo que indica `ingestion_time()` si el valor `rhs` del cursor de base de datos del registro procede después del valor del cursor de base de datos.

* [cursor_before_or_at(rhs:string):](../query/cursorbeforeoratfunction.md)esta función especial se puede utilizar en los registros de tabla que tienen habilitada la [directiva IngestionTime.](ingestiontime-policy.md) Devuelve un valor escalar de `bool` tipo que indica `ingestion_time()` si el valor `rhs` del cursor de base de datos del registro procede después del valor del cursor de base de datos.

Las dos`cursor_after` funciones `cursor_before_or_at`especiales ( y ) también tienen un efecto secundario: cuando se utilizan, Kusto emitirá el **valor actual del cursor de base de** datos al conjunto de `@ExtendedProperties` resultados de la consulta. El nombre de propiedad `Cursor`del cursor es `string`, y su valor es un único archivo . Por ejemplo:

```json
{"Cursor" : "636040929866477946"}
```

## <a name="restrictions"></a>Restricciones

Los cursores de base de datos solo se pueden usar con tablas para las que se ha habilitado la [directiva IngestionTime.](ingestiontime-policy.md) Cada registro de una tabla de este tipo está asociado con el valor del cursor de base de datos que estaba en vigor cuando se ingirió el registro y, por lo tanto, se puede utilizar la función [ingestion_time().](../query/ingestiontimefunction.md)

El objeto cursor de base de datos no contiene ningún valor significativo a menos que la base de datos tenga al menos una tabla que tenga definida una [directiva IngestionTime.](ingestiontime-policy.md)
Además, solo se garantiza que este valor se actualiza según sea necesario por el historial de ingesta en dichas tablas y las consultas ejecutan esas tablas de referencia. Es posible que se actualice o no en otros casos.

El proceso de ingesta primero confirma los datos (para que estén disponibles para realizar consultas) y solo después asigna un valor de cursor real a cada registro. Esto significa que si se intenta consultar datos inmediatamente después de la finalización de la ingesta mediante un cursor de base de datos, es posible que los resultados aún no incorporen los últimos registros agregados, porque aún no se les ha asignado el valor del cursor. De forma similar, recuperar el valor actual del cursor de base de datos repetidamente podría devolver el mismo valor (incluso si la ingesta se realizó en el medio) porque solo la confirmación del cursor actualizará su valor.

## <a name="example-processing-of-records-exactly-once"></a>Ejemplo: Procesamiento de registros exactamente Una vez

Suponga `Employees` la `[Name, Salary]`tabla con el esquema .
Para procesar continuamente nuevos registros a medida que se ingenian en la tabla, utilice el siguiente procedimiento:

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
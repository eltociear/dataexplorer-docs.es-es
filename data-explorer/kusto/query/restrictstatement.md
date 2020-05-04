---
title: 'Instrucción Restrict: Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe la instrucción Restrict en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 094cec5b467c35eb9dbeeb756362bd13c77873ce
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737783"
---
# <a name="restrict-statement"></a>Instrucción Restrict

::: zone pivot="azuredataexplorer"

La instrucción Restrict limita el conjunto de entidades de tabla o vista que son visibles para las instrucciones de consulta que las siguen. Por ejemplo, en una base de datos que incluye dos`A`tablas `B`(,), la aplicación puede evitar que el resto de la consulta `B` obtenga acceso y solo "vea" una forma limitada `A` de tabla mediante una vista.

El escenario principal de la instrucción Restrict es para las aplicaciones de nivel intermedio que aceptan consultas de usuarios y desean aplicar un mecanismo de seguridad de nivel de fila a través de esas consultas. La aplicación de nivel intermedio puede prefijar la consulta del usuario con un **modelo lógico**, un conjunto de instrucciones Let que definen vistas que restringen el acceso del usuario a los datos `T | where UserId == "..."`(por ejemplo,). A medida que se agrega la última instrucción, solo se restringe el acceso del usuario al modelo lógico.

**Sintaxis**

`restrict``access` `,` [EntitySpecifier [...]]*EntitySpecifier* `to` `(``)`

Donde *EntitySpecifier* es uno de los siguientes:
* Identificador definido por una instrucción Let como una vista tabular.
* Una referencia de tabla (similar a la usada por una instrucción Union).
* Patrón definido por una declaración de patrón.

Todas las tablas, vistas tabulares o patrones que no se especifican en la instrucción Restrict se convierten en "invisibles" en el resto de la consulta. 

**Notas**

La instrucción Restrict puede usarse para restringir el acceso a las entidades en otra base de datos o clúster (no se admiten caracteres comodín en los nombres de clúster).

**Argumentos**

La instrucción Restrict puede obtener uno o varios parámetros que definan la restricción permisiva durante la resolución de nombres de la entidad. La entidad puede ser:
- [instrucción Let que](./letstatement.md) aparece antes `restrict` de la instrucción. 

```kusto
// Limit access to 'Test' let statement only
let Test = () { print x=1 };
restrict access to (Test);
```

- [Tablas](../management/tables.md) o [funciones](../management/functions.md) que se definen en los metadatos de la base de datos.

```kusto
// Assuming the database that the query uses has table Table1 and Func1 defined in the metadata, 
// and other database 'DB2' has Table2 defined in the metadata
 
restrict access to (database().Table1, database().Func1, database('DB2').Table2);
```

- Patrones de caracteres comodín que pueden coincidir con múltiplos de [instrucciones Let](./letstatement.md) o tablas o funciones  

```kusto
let Test1 = () { print x=1 };
let Test2 = () { print y=1 };
restrict access to (*);
// Now access is restricted to Test1, Test2 and no tables/functions are accessible.

// Assuming the database that the query uses has table Table1 and Func1 defined in the metadata.
// Assuming that database 'DB2' has table Table2 and Func2 defined in the metadata
restricts access to (database().*);
// Now access is restricted to all tables/functions of the current database ('DB2' is not accessible).

// Assuming the database that the query uses has table Table1 and Func1 defined in the metadata.
// Assuming that database 'DB2' has table Table2 and Func2 defined in the metadata
restricts access to (database('DB2').*);
// Now access is restricted to all tables/functions of the database 'DB2'
```


**Ejemplos**

En el ejemplo siguiente se muestra cómo una aplicación de nivel medio puede anteponer una consulta de usuario con un modelo lógico que impide que el usuario Consulte los datos de otros usuarios.

```kusto
// Assume the database has a single table, UserData,
// with a column called UserID and other columns that hold
// per-user private information.
//
// The middle-tier application generates the following statements.
// Note that "username@domain.com" is something the middle-tier application
// derives per-user as it authenticates the user.
let RestrictedData = view () { Data | where UserID == "username@domain.com" };
restrict access to (RestrictedData);
// The rest of the query is something that the user types.
// This part can only reference RestrictedData; attempting to reference Data
// will fail.
RestrictedData | summarize IrsLovesMe=sum(Salary) by Year, Month
```

```kusto
// Restricting access to Table1 in the current database (database() called without parameters)
restrict access to (database().Table1);
Table1 | count

// Restricting acess to Table1 in the current database and Table2 in database 'DB2'
restrict access to (database().Table1, database('DB2').Table2);
union 
    (Table1),
    (database('DB2').Table2))
| count

// Restricting access to Test statement only
let Test = () { range x from 1 to 10 step 1 };
restrict access to (Test);
Test
 
// Assume that there is a table called Table1, Table2 in the database
let View1 = view () { Table1 | project Column1 };
let View2 = view () { Table2 | project Column1, Column2 };
restrict access to (View1, View2);
 
// When those statements appear before the command - the next works
let View1 = view () { Table1 | project Column1 };
let View2 = view () { Table2 | project Column1, Column2 };
restrict access to (View1, View2);
View1 |  count
 
// When those statements appear before the command - the next access is not allowed
let View1 = view () { Table1 | project Column1 };
let View2 = view () { Table2 | project Column1, Column2 };
restrict access to (View1, View2);
Table1 |  count
```

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

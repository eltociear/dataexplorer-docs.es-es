---
title: 'Directiva de actualización: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de actualización en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 95952a6f4e7a8c0d1a5b4207742e15873eb44c91
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519540"
---
# <a name="update-policy"></a>Actualización de directiva

La directiva de [actualización](updatepolicy.md) es un objeto de directiva de nivel de tabla para ejecutar automáticamente una consulta e ingerir sus resultados cuando los datos se ingiren en otra tabla.

## <a name="show-update-policy"></a>Mostrar directiva de actualización

Este comando devuelve la directiva de actualización de la tabla `*` especificada o todas las tablas de la base de datos predeterminada si se utiliza como nombre de tabla.

**Sintaxis**

* `.show``table` *TableName* `policy``update`
* `.show``table` *DatabaseName*`.`*TableName* `policy``update`
* `.show` `table` `*` `policy` `update`

**Devuelve**

Este comando devuelve una tabla que tiene un registro por tabla, con las siguientes columnas:

|Columna    |Tipo    |Descripción                                                                                                                                                           |
|----------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|EntityName|`string`|El nombre de la entidad en la que se define la directiva de actualización                                                                                                                |
|Directivas  |`string`|Matriz JSON que indica todas las políticas de actualización definidas para la entidad, formateadas como objeto de política de [actualización](updatepolicy.md#the-update-policy-object)|

**Ejemplo**

```kusto
.show table DerivedTableX policy update 
```

|EntityName        |Directivas                                                                                                                                    |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|[TestDB]. [DerivedTableX]|["IsEnabled": true, "Source": "MyTableX","Query": "MyUpdateFunction()","IsTransactional": false,"PropagateIngestionProperties": false-]|

## <a name="alter-update-policy"></a>Modificar la política de actualización

Este comando establece la directiva de actualización de la tabla especificada.

**Sintaxis**

* `.alter``table` *TableName* `policy` TableName `update` *ArrayOfUpdatePolicyObjects*
* `.alter``table` *DatabaseName*`.`*TableName* `policy` `update` *ArrayOfUpdatePolicyObjects*

*ArrayOfUpdatePolicyObjects* es una matriz JSON que tiene cero o más objetos de política de actualización definidos.

**Notas**

1. Se recomienda que se utilice una `Query` función almacenada para la propiedad del objeto de directiva de actualización.
   Esto facilita la modificación solo de la definición de función en lugar de todo el objeto de política.

2. Las siguientes validaciones se realizan en la directiva de `IsEnabled` actualización `true`cuando se establece (en caso de que se establezca en ):
    1. `Source`: La tabla debe existir en la base de datos de destino.
    2. `Query`: 
        * El esquema definido por el esquema debe coincidir con el de la tabla de destino. 
        * La consulta debe `source` hacer referencia a la tabla de la directiva de actualización. Definir una consulta de *not* directiva de actualización que `AllowUnreferencedSourceTable=true` no hace referencia al origen es posible estableciendo en las propiedades with (consulte el ejemplo siguiente), pero generalmente no se recomienda debido a razones de rendimiento (implica que para cada ingesta a la tabla de origen, *todos los* registros de una tabla diferente se consideran para la ejecución de la directiva de actualización).
    3. La directiva no se da como resultado con un ciclo que se crea en la cadena de directivas de actualización en la base de datos de destino.
    4. En `IsTransactional` caso de `true` `TableAdmin` que se `Source` establezca en , los permisos también se verifican (la tabla de origen).
  
3. Asegúrese de probar la directiva/función de actualización para el rendimiento antes de aplicarla para ejecutarse en cada ingesta a la tabla de origen - consulte [aquí](updatepolicy.md#testing-an-update-policys-performance-impact).

**Devuelve**

El comando establece el objeto de política de actualización de la tabla (reemplazando cualquier directiva actual definida, si existe) y, a continuación, devuelve la salida del comando de política de actualización TABLE de [la tabla .show](#show-update-policy) correspondiente.

**Ejemplo**

```kusto
// Creating function that will be used for update
.create function 
MyUpdateFunction()
{
    MyTableX
    | where ColumnA == 'some-string'
    | summarize MyCount=count() by ColumnB, Key=ColumnC
    | join (OtherTable | project OtherColumnZ, Key=OtherColumnC) on Key
    | project ColumnB, ColumnZ=OtherColumnZ, Key, MyCount
}

// Creating the target table (in case it doesn't already exist)
.set-or-append DerivedTableX <| MyUpdateFunction() | limit 0

// Use update policy on table DerivedTableX
.alter table DerivedTableX policy update
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyUpdateFunction()", "IsTransactional": false, "PropagateIngestionProperties": false}]'
```

- Cuando se produce una ingesta a `MyTableX`la tabla de origen (en este caso), se crean 1 o más extensiones (particiones de datos) en esa tabla.
- El `Query` que se define en el objeto `MyUpdateFunction()`de directiva de actualización (en este caso ) solo se ejecutará en esas extensiones y no se ejecutará en toda la tabla.
  - Este "scoping" se realiza internamente y automáticamente, no `Query`debe ser manejado al definir el archivo .
  - Solo se tendrán en cuenta los registros recién ingeridos (diferentes en cada operación `DerivedTableX`de ingesta) al ingerir en la tabla derivada (en este caso).


```kusto
// The following example will throw an error for not referencing the source table in the update policy query
// The policy's source table is MyTableX, whereas the query only references MyOtherTable. 
.alter table DerivedTableX policy update
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyOtherTable", "IsTransactional": false, "PropagateIngestionProperties": false}]'

// Adding the following properties will suppress the error (but is not recommended)
.alter table DerivedTableX policy update with (AllowUnreferencedSourceTable=true)
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyOtherTable", "IsTransactional": false, "PropagateIngestionProperties": false}]'

```

## <a name="alter-merge-table-table-policy-update"></a>Actualización de la directiva TABLE de la tabla .alter-merge

Este comando modifica la directiva de actualización de la tabla especificada.

**Sintaxis**

* `.alter-merge``table` *TableName* `policy` TableName `update` *ArrayOfUpdatePolicyObjects*
* `.alter-merge``table` *DatabaseName*`.`*TableName* `policy` `update` *ArrayOfUpdatePolicyObjects*

*ArrayOfUpdatePolicyObjects* es una matriz JSON que tiene cero o más objetos de política de actualización definidos.

**Notas**

1. Se recomienda que se utilicen funciones almacenadas para la implementación masiva de la propiedad query del objeto de directiva de actualización. Esto facilita la modificación solo de la definición de función en lugar de todo el objeto de política.

2. Las mismas validaciones realizadas en la `alter` directiva de `alter-merge` actualización en caso de un comando se realizan para un comando.

**Devuelve**

El comando anexa al objeto de política de actualización de la tabla (reemplazando cualquier directiva actual definida, si existe) y, a continuación, devuelve la salida del comando de política de actualización TABLE de [la tabla .show](#show-update-policy) correspondiente.

**Ejemplo**

```kusto
.alter-merge table DerivedTableX policy update 
@'[{"IsEnabled": true, "Source": "MyTableY", "Query": "MyUpdateFunction()", "IsTransactional": false}]'  
``` 

## <a name="delete-table-table-policy-update"></a>Actualización de la directiva TABLE de la tabla .delete

Elimina la directiva de actualización de la tabla especificada.

**Sintaxis**

* `.delete``table` *TableName* `policy``update`
* `.delete``table` *DatabaseName*`.`*TableName* `policy``update`

**Devuelve**

El comando elimina el objeto de política de actualización de la tabla y, a continuación, devuelve la salida del comando de política de actualización TABLE de [la tabla .show](#show-update-policy) correspondiente.

**Ejemplo**

```kusto
.delete table DerivedTableX policy update 
```
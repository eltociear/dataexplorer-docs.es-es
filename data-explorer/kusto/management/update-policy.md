---
title: 'Administración de directivas de actualización de Kusto: Azure Explorador de datos'
description: En este artículo se describe la Directiva de actualización en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 5974909b6f5e40a977f935319972e7477a2a5755
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294379"
---
# <a name="update-policy"></a>Actualización de directiva

La [Directiva de actualización](updatepolicy.md) es un objeto de directiva de nivel de tabla que ejecuta automáticamente una consulta y, a continuación, ingeri los resultados cuando se ingestan datos en otra tabla.

## <a name="show-update-policy"></a>Mostrar Directiva de actualización

Este comando devuelve la Directiva de actualización de la tabla especificada o todas las tablas de la base de datos predeterminada si `*` se usa como nombre de tabla.

**Sintaxis**

* `.show``table` *TableName* TableName `policy``update`
* `.show``table` *NombreDeBaseDeDatos* `.` *TableName* TableName `policy``update`
* `.show` `table` `*` `policy` `update`

**Devuelve**

Este comando devuelve una tabla que tiene un registro por tabla.

|Columna    |Tipo    |Descripción                                                                                                                                                           |
|----------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|EntityName|`string`|El nombre de la entidad en la que se define la Directiva de actualización                                                                                                                |
|Directivas  |`string`|Una matriz JSON que indica todas las directivas de actualización definidas para la entidad, con el formato de [objeto de directiva de actualización](updatepolicy.md#the-update-policy-object)|

**Ejemplo**

```kusto
.show table DerivedTableX policy update 
```

|EntityName        |Directivas                                                                                                                                    |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|[TestDB]. [DerivedTableX]|[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyUpdateFunction ()", "IsTransactional": false, "PropagateIngestionProperties": false}]|

## <a name="alter-update-policy"></a>Modificar Directiva de actualización

Este comando establece la Directiva de actualización de la tabla especificada.

**Sintaxis**

* `.alter``table` *TableName* `policy` TableName `update` *ArrayOfUpdatePolicyObjects*
* `.alter``table` *NombreDeBaseDeDatos* `.` *TableName* `policy` `update` *ArrayOfUpdatePolicyObjects*

*ArrayOfUpdatePolicyObjects* es una matriz JSON que tiene cero o más objetos de directiva de actualización definidos.

> [!NOTE]
> * Use una función almacenada para la `Query` propiedad del objeto de directiva de actualización.
   Solo tendrá que modificar la definición de función, en lugar de todo el objeto de directiva.
> * Si `IsEnabled` se establece en `true` , se realizarán las validaciones siguientes en la Directiva de actualización a medida que se establece:
>    * `Source`: Comprueba que la tabla existe en la base de datos de destino.
>    * `Query` 
>        * Comprueba que el esquema definido por el esquema coincide con el de la tabla de destino.
>        * Comprueba que la consulta hace referencia a la `source` tabla de la Directiva de actualización. 
        La definición de una consulta de directiva de actualización que *no haga referencia al origen* es posible mediante `AllowUnreferencedSourceTable=true` el establecimiento de en las propiedades *with* (vea el ejemplo siguiente), pero no se recomienda debido a problemas de rendimiento. Para cada ingesta en la tabla de origen, se tienen en cuenta *todos* los registros de una tabla diferente para la ejecución de la Directiva de actualización.
 >       * Comprueba que la Directiva no da lugar a la creación de un ciclo en la cadena de directivas de actualización de la base de datos de destino.
 > * Si `IsTransactional` está establecido en `true` , comprueba que los `TableAdmin` permisos también se comprueban con `Source` (la tabla de origen).
 > * Pruebe la Directiva de actualización o la función para el rendimiento, antes de aplicarla para que se ejecute en cada ingesta en la tabla de origen. Para obtener más información, consulte probar el impacto en el [rendimiento de una directiva de actualización](updatepolicy.md#testing-an-update-policys-performance-impact).

**Devuelve**

El comando establece el objeto de directiva de actualización de la tabla, invalidando cualquier directiva actual y, a continuación, devuelve el resultado del comando [. Mostrar Directiva de actualización de tabla de tabla](#show-update-policy) correspondiente.

**Ejemplo**

```kusto
// Create a function that will be used for update
.create function 
MyUpdateFunction()
{
    MyTableX
    | where ColumnA == 'some-string'
    | summarize MyCount=count() by ColumnB, Key=ColumnC
    | join (OtherTable | project OtherColumnZ, Key=OtherColumnC) on Key
    | project ColumnB, ColumnZ=OtherColumnZ, Key, MyCount
}

// Create the target table (if it doesn't already exist)
.set-or-append DerivedTableX <| MyUpdateFunction() | limit 0

// Use update policy on table DerivedTableX
.alter table DerivedTableX policy update
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyUpdateFunction()", "IsTransactional": false, "PropagateIngestionProperties": false}]'
```

* Cuando se produce una ingesta en la tabla de origen, en este caso, `MyTableX` se crean una o varias extensiones (particiones de datos) en esa tabla.
* El `Query` que se define en el objeto de directiva de actualización, en este caso ' MyUpdateFunction (), solo se ejecutará en esas extensiones y no se ejecutará en toda la tabla.
  * "Ámbito" se realiza interna y automáticamente, y no debe administrarse al definir `Query` .
  * Solo se tendrán en cuenta los registros recién incorporados, que son diferentes en cada operación de ingesta, cuando se ingestan a la `DerivedTableX` tabla derivada.

```kusto
// The following example will throw an error for not referencing the source table in the update policy query
// The policy's source table is MyTableX, whereas the query only references MyOtherTable. 
.alter table DerivedTableX policy update
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyOtherTable", "IsTransactional": false, "PropagateIngestionProperties": false}]'

// Adding the following properties will suppress the error (but is not recommended)
.alter table DerivedTableX policy update with (AllowUnreferencedSourceTable=true)
@'[{"IsEnabled": true, "Source": "MyTableX", "Query": "MyOtherTable", "IsTransactional": false, "PropagateIngestionProperties": false}]'

```

## <a name="alter-merge-table-table-policy-update"></a>. Alter-Merge tabla Table Update de la Directiva

Este comando modifica la Directiva de actualización de la tabla especificada.

**Sintaxis**

* `.alter-merge``table` *TableName* `policy` TableName `update` *ArrayOfUpdatePolicyObjects*
* `.alter-merge``table` *NombreDeBaseDeDatos* `.` *TableName* `policy` `update` *ArrayOfUpdatePolicyObjects*

*ArrayOfUpdatePolicyObjects* es una matriz JSON que tiene cero o más objetos de directiva de actualización definidos.

> [!NOTE]
> * Use funciones almacenadas para la implementación masiva de la propiedad de consulta del objeto de directiva de actualización. 
     Solo tendrá que modificar la definición de función en lugar de todo el objeto de directiva.
> * Las validaciones son las mismas que las que se realizan en un `alter` comando.

**Devuelve**

El comando anexa al objeto de directiva de actualización de la tabla, invalidando cualquier directiva actual y, a continuación, devuelve el resultado del comando [. Mostrar Directiva de actualización de tabla de tabla](#show-update-policy) correspondiente.

**Ejemplo**

```kusto
.alter-merge table DerivedTableX policy update 
@'[{"IsEnabled": true, "Source": "MyTableY", "Query": "MyUpdateFunction()", "IsTransactional": false}]'  
``` 

## <a name="delete-table-table-policy-update"></a>. eliminar actualización de directiva de tabla de tabla

Elimina la Directiva de actualización de la tabla especificada.

**Sintaxis**

* `.delete``table` *TableName* TableName `policy``update`
* `.delete``table` *NombreDeBaseDeDatos* `.` *TableName* TableName `policy``update`

**Devuelve**

El comando elimina el objeto de directiva de actualización de la tabla y, a continuación, devuelve el resultado del comando correspondiente [. Mostrar la Directiva de actualización de tabla de tabla](#show-update-policy) .

**Ejemplo**

```kusto
.delete table DerivedTableX policy update 
```

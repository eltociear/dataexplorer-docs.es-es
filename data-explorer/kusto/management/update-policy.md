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
ms.openlocfilehash: 320824b4f614ea809141167c1284282b1ef641cf
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616565"
---
# <a name="update-policy"></a>Actualización de directiva

La [Directiva de actualización](updatepolicy.md) es un objeto de directiva de nivel de tabla para ejecutar automáticamente una consulta y ingerir los resultados cuando se introducen datos en otra tabla.

## <a name="show-update-policy"></a>Mostrar Directiva de actualización

Este comando devuelve la Directiva de actualización de la tabla especificada o todas las tablas de la base de `*` datos predeterminada si se usa como nombre de tabla.

**Sintaxis**

* `.show``table` *TableName* TableName `policy``update`
* `.show``table` *DatabaseName*NombreDeBaseDeDatos`.`*TableName* TableName `policy``update`
* `.show` `table` `*` `policy` `update`

**Devuelve**

Este comando devuelve una tabla que tiene un registro por tabla, con las columnas siguientes:

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

* `.alter``table` *TableName* TableName `policy` *ArrayOfUpdatePolicyObjects* ArrayOfUpdatePolicyObjects `update`
* `.alter``table` *NombreDeBaseDeDatos* `update` *ArrayOfUpdatePolicyObjects* *TableName* TableName ArrayOfUpdatePolicyObjects`.` `policy`

*ArrayOfUpdatePolicyObjects* es una matriz JSON que tiene cero o más objetos de directiva de actualización definidos.

**Notas**

1. Se recomienda usar una función almacenada para la `Query` propiedad del objeto de directiva de actualización.
   Esto facilita la modificación de la definición de función en lugar de todo el objeto de directiva.

2. Las validaciones siguientes se realizan en la Directiva de actualización cuando se establece (en caso `IsEnabled` de que se establezca en `true`):
    1. `Source`: La tabla debe existir en la base de datos de destino.
    2. `Query`: 
        * El esquema definido por el esquema debe coincidir con el de la tabla de destino. 
        * La consulta debe hacer referencia `source` a la tabla de la Directiva de actualización. La definición de una consulta de directiva de actualización que *no haga referencia al* origen `AllowUnreferencedSourceTable=true` es posible mediante el establecimiento de en las propiedades de con (vea el ejemplo siguiente), pero generalmente no se recomienda por motivos de rendimiento (implica que, para cada ingesta en la tabla de origen, se tienen en cuenta *todos* los registros de una tabla diferente para la ejecución de la Directiva de actualización).
    3. La Directiva no resulta en la creación de un ciclo en la cadena de directivas de actualización en la base de datos de destino.
    4. En caso `IsTransactional` de que `true`se establezca `TableAdmin` en, los permisos `Source` se comprueban también en (la tabla de origen).
  
3. Asegúrese de probar la Directiva o la función de actualización para ver el rendimiento antes de aplicarlo para que se ejecute en cada ingesta en la tabla de origen. consulte [aquí](updatepolicy.md#testing-an-update-policys-performance-impact).

**Devuelve**

El comando establece el objeto de directiva de actualización de la tabla (invalidando las directivas actuales definidas, si hay alguna) y, a continuación, devuelve el resultado del comando correspondiente [. Mostrar Directiva de actualización de tabla de tabla](#show-update-policy) .

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

- Cuando se produce una ingesta en la tabla de origen ( `MyTableX`en este caso), se crean 1 o más extensiones (particiones de datos) en esa tabla.
- El `Query` que se define en el objeto de directiva de actualización (en `MyUpdateFunction()`este caso) solo se ejecutará en esas extensiones y no se ejecutará en toda la tabla.
  - Este "ámbito" se realiza interna y automáticamente, no se debe administrar al definir `Query`.
  - Solo se tomarán en consideración los registros recién incorporados (diferentes en cada operación de ingesta) al ingesta en la tabla derivada (en `DerivedTableX`este caso).


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

* `.alter-merge``table` *TableName* TableName `policy` *ArrayOfUpdatePolicyObjects* ArrayOfUpdatePolicyObjects `update`
* `.alter-merge``table` *NombreDeBaseDeDatos* `update` *ArrayOfUpdatePolicyObjects* *TableName* TableName ArrayOfUpdatePolicyObjects`.` `policy`

*ArrayOfUpdatePolicyObjects* es una matriz JSON que tiene cero o más objetos de directiva de actualización definidos.

**Notas**

1. Se recomienda usar funciones almacenadas para la implementación masiva de la propiedad de consulta del objeto de directiva de actualización. Esto facilita la modificación de la definición de función en lugar de todo el objeto de directiva.

2. Las mismas validaciones realizadas en la Directiva de actualización en caso de que `alter` se realice un comando para `alter-merge` un comando.

**Devuelve**

El comando anexa al objeto de directiva de actualización de la tabla (invalidando las directivas actuales definidas, si hay alguna) y, a continuación, devuelve el resultado del comando correspondiente [. Mostrar Directiva de actualización de tabla de tabla](#show-update-policy) .

**Ejemplo**

```kusto
.alter-merge table DerivedTableX policy update 
@'[{"IsEnabled": true, "Source": "MyTableY", "Query": "MyUpdateFunction()", "IsTransactional": false}]'  
``` 

## <a name="delete-table-table-policy-update"></a>. eliminar actualización de directiva de tabla de tabla

Elimina la Directiva de actualización de la tabla especificada.

**Sintaxis**

* `.delete``table` *TableName* TableName `policy``update`
* `.delete``table` *DatabaseName*NombreDeBaseDeDatos`.`*TableName* TableName `policy``update`

**Devuelve**

El comando elimina el objeto de directiva de actualización de la tabla y, a continuación, devuelve el resultado del comando correspondiente [. Mostrar la Directiva de actualización de tabla de tabla](#show-update-policy) .

**Ejemplo**

```kusto
.delete table DerivedTableX policy update 
```
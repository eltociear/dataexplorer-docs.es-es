---
title: 'Crear y modificar tablas externas de SQL: Azure Explorador de datos'
description: En este artículo se describe cómo crear y modificar tablas externas de SQL.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 235c68a8a04fd76dd3a9e25abac63db09e00919a
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863343"
---
# <a name="create-and-alter-external-sql-tables"></a>Crear y modificar tablas externas de SQL

Crea o modifica una tabla SQL externa en la base de datos en la que se ejecuta el comando.  

## <a name="syntax"></a>Sintaxis

( `.create`  |  `.alter` ) `external` `table` *TableName* ([ColumnName: columnType],...)  
`kind` `=` `sql`  
`table``=` *SqlTableName*  
`(`*SqlServerConnectionString*`)`  
[ `with` `(` [ `docstring` `=` *Documentación*] [ `,` `folder` `=` *nombrecarpeta*], *property_name* `=` *valor* `,` ... `)` ]

## <a name="parameters"></a>Parámetros

* *TableName* : nombre de tabla externa. Debe seguir las reglas para [los nombres de entidad](../query/schema-entities/entity-names.md). Una tabla externa no puede tener el mismo nombre que una tabla normal de la misma base de datos.
* *SqlTableName* : el nombre de la tabla SQL.
* *SqlServerConnectionString* : la cadena de conexión a SQL Server. Puede ser uno de los métodos siguientes: 
  * *Autenticación integrada de AAD* ( `Authentication="Active Directory Integrated"` ): el usuario o la aplicación se autentica a través de AAD a Kusto y, a continuación, se usa el mismo token para tener acceso al extremo de red SQL Server.
  * *Autenticación de nombre de usuario/contraseña* ( `User ID=...; Password=...;` ). Si la tabla externa se utiliza para la [exportación continua](data-export/continuous-data-export.md), la autenticación se debe realizar mediante este método. 

> [!WARNING]
> Las cadenas de conexión y las consultas que incluyen información confidencial se deben ofuscar para que se omitan en cualquier seguimiento de Kusto. Para obtener más información, vea [literales de cadena ofuscados](../query/scalar-data-types/string.md#obfuscated-string-literals).

## <a name="optional-properties"></a>Propiedades opcionales

| Propiedad            | Tipo            | Descripción                          |
|---------------------|-----------------|---------------------------------------------------------------------------------------------------|
| `folder`            | `string`        | La carpeta de la tabla.                  |
| `docString`         | `string`        | Cadena que documenta la tabla.      |
| `firetriggers`      | `true`/`false`  | Si `true` , indica al sistema de destino que active los desencadenadores de inserción definidos en la tabla SQL. El valor predeterminado es `false`. (Para obtener más información, vea [Bulk Insert](https://msdn.microsoft.com/library/ms188365.aspx) y [System. Data. SqlClient. SqlBulkCopy).](https://msdn.microsoft.com/library/system.data.sqlclient.sqlbulkcopy(v=vs.110).aspx) |
| `createifnotexists` | `true`/ `false` | Si es `true` , se creará la tabla SQL de destino si aún no existe; `primarykey` en este caso, se debe proporcionar la propiedad para indicar la columna de resultados que es la clave principal. El valor predeterminado es `false`.  |
| `primarykey`        | `string`        | Si `createifnotexists` es `true` , el nombre de la columna resultante se usará como clave principal de la tabla SQL si este comando lo crea.                  |

> [!NOTE]
> * Si la tabla existe, `.create` se producirá un error en el comando. `.alter`Se usa para modificar las tablas existentes. 
> * No se admite la modificación del esquema o el formato de una tabla SQL externa. 

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md) para `.create` y [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para `.alter` . 
 
**Ejemplo** 

```kusto
.create external table ExternalSql (x:long, s:string) 
kind=sql
table=MySqlTable
( 
   h@'Server=tcp:myserver.database.windows.net,1433;Authentication=Active Directory Integrated;Initial Catalog=mydatabase;'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables", 
   createifnotexists = true,
   primarykey = x,
   firetriggers=true
)  
```

**Salida**

| TableName   | TableType | Carpeta         | DocString | Propiedades                            |
|-------------|-----------|----------------|-----------|---------------------------------------|
| ExternalSql | Sql       | ExternalTables | Documentos      | {<br>  "TargetEntityKind": "sqltable'",<br>  "TargetEntityName": "MySqlTable",<br>  "TargetEntityConnectionString": "Server = TCP:myserver. Database. Windows. net, 1433; Authentication = Active Directory integrado; Initial Catalog = base de datos; ",<br>  "FireTriggers": true,<br>  "CreateIfNotExists": true,<br>  "PrimaryKey": "x"<br>} |

## <a name="querying-an-external-table-of-type-sql"></a>Consultar una tabla externa de tipo SQL 

Se admite la consulta de una tabla SQL externa. Consulte [consultar tablas externas](../../data-lake-query-data.md). 

> [!Note]
> La implementación de consultas de tabla externa de SQL ejecutará una ' SELECT * ' completa (o seleccione las columnas pertinentes) de la tabla SQL. El resto de la consulta se ejecutará en el lado de Kusto. 

Considere la siguiente consulta de tabla externa: 

```kusto
external_table('MySqlExternalTable') | count
```

Kusto ejecutará una consulta ' SELECT * FROM TABLE ' en la base de datos SQL, seguida de un recuento en el lado de Kusto. En tales casos, se espera que el rendimiento sea mejor si se escribe directamente en T-SQL (' SELECT COUNT (1) FROM TABLE ') y se ejecuta con el [complemento sql_request](../query/sqlrequestplugin.md), en lugar de usar la función de tabla externa. Del mismo modo, los filtros no se insertan en la consulta SQL.  

Use la tabla externa para consultar la tabla SQL cuando la consulta requiera leer toda la tabla (o columnas relevantes) para su posterior ejecución en el lado de Kusto. Cuando una consulta SQL se pueda optimizar en T-SQL, use el [complemento sql_request](../query/sqlrequestplugin.md).

## <a name="next-steps"></a>Pasos siguientes

* [Comandos de control general de tabla externa](externaltables.md)
* [Crear y modificar tablas externas en Azure Storage o Azure Data Lake](external-tables-azurestorage-azuredatalake.md)

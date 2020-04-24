---
title: 'Exportar datos a SQL: Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe Exportar datos a SQL en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7deeb3868800f00a828eb09cc605e4d7e5227032
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521631"
---
# <a name="export-data-to-sql"></a>Exportar datos a SQL

Exportar datos a SQL le permite ejecutar una consulta y enviar sus resultados a una tabla de una base de datos SQL, como una base de datos SQL hospedada por el servicio Azure SQL Database.

**Sintaxis**

`.export`[`async` `to` `sql` ] *SqlTableName* *SqlConnectionString* `with` `(`[ *PropertyName* `=` *PropertyValue*`,`... `)`] `<|` *Consulta*

Donde:
* *async*: Command se ejecuta en modo asincrónico (opcional).
* *SqlTableName* Nombre de tabla de base de datos SQL donde se insertan los datos.
  Para protegerse contra ataques de inyección, este nombre está restringido.
* *SqlConnectionString* es `string` un literal `ADO.NET` que sigue el formato de cadena de conexión y describe el extremo SQL y la base de datos a la que se conecta. Por motivos de seguridad, la cadena de conexión está restringida.
* *PropertyName*, *PropertyValue* son pares de un nombre (identificador) y un valor (literal de cadena).

Propiedades:

|Nombre               |Valores           |Descripción|
|-------------------|-----------------|-----------|
|`firetriggers`     |`true` o `false`|Si `true`, indica al sistema de destino que active los desencadenadores INSERT definidos en la tabla SQL. El valor predeterminado es `false`. (Para obtener más información, vea [BULK INSERT](https://msdn.microsoft.com/library/ms188365.aspx) y [System.Data.SqlClient.SqlBulkCopy](https://msdn.microsoft.com/library/system.data.sqlclient.sqlbulkcopy(v=vs.110).aspx))|
|`createifnotexists`|`true` o `false`|Si `true`, se creará la tabla SQL de destino si aún no existe; la `primarykey` propiedad debe proporcionarse en este caso para indicar la columna de resultados que es la clave principal. El valor predeterminado es `false`.|
|`primarykey`       |                 |Si `createifnotexists` `true`es , indica el nombre de la columna en el resultado que se usará como clave principal de la tabla SQL si se crea mediante este comando.|
|`persistDetails`   |`bool`           |Indica que el comando debe conservar `async` sus resultados (consulte la marca). El valor `true` predeterminado es en ejecuciones asincrónicas, pero se puede desactivar si el autor de la llamada no requiere los resultados). El valor `false` predeterminado es en las ejecuciones sincrónicas, pero se puede activar. |
|`token`            |`string`         |El token de acceso de AAD que Kusto reenviará al punto de conexión SQL para la autenticación. Cuando se establece, la cadena de conexión `Authentication` `User ID`SQL `Password`no debe incluir información de autenticación como , , o .|

## <a name="limitations-and-restrictions"></a>Limitaciones y restricciones

Existen varias limitaciones y restricciones al exportar datos a una base de datos SQL:

1. Kusto es un servicio en la nube, por lo que la cadena de conexión debe apuntar a una base de datos accesible desde la nube. (En particular, no se puede exportar a una base de datos local, ya que no es accesible desde la nube pública.)

2. Kusto admite la autenticación integrada de Active Directory cuando`aaduser=` `aadapp=`la entidad de seguridad de llamada es una entidad de seguridad de Azure Active Directory ( o ).
   Como alternativa, Kusto también admite el suministro de las credenciales para la base de datos SQL como parte de la cadena de conexión. No se admiten otros métodos de autenticación. La identidad que se presenta a la base de datos SQL siempre emana del llamador de comandos, no de la identidad del servicio Kusto en sí.

3. Si existe la tabla de destino de la base de datos SQL, debe coincidir con el esquema de resultados de la consulta. Tenga en cuenta que en algunos casos (como Azure SQL Database) esto significa que la tabla tiene una columna marcada como columna de identidad.

4. La exportación de grandes volúmenes de datos puede tardar mucho tiempo. Se recomienda establecer la tabla SQL de destino para un registro mínimo durante la importación masiva.
   Consulte Motor de base de datos de SQL ServerSQL Server Database Engine > ... características de base de datos [> > importación y exportación masiva de datos](https://msdn.microsoft.com/library/ms190422.aspx).

5. La exportación de datos se realiza mediante la copia masiva de SQL y no proporciona ninguna garantía transaccional en la base de datos SQL de destino. Consulte Operaciones de [transacción y copia masiva](https://docs.microsoft.com/dotnet/framework/data/adonet/sql/transaction-and-bulk-copy-operations) para obtener más detalles.

6. El nombre de la tabla SQL está restringido a un nombre`_`que consta`.`de letras, dígitos, espacios, guiones bajos ( ), puntos ( ) y guiones (`-`).

7. La cadena de conexión SQL `Persist Security Info` está restringida de `false` `Encrypt` la siguiente `true`manera: se establece explícitamente en , se establece en , y `Trust Server Certificate` se establece en `false`.

8. La propiedad de clave principal de la columna se puede especificar al crear una nueva tabla SQL. Si la columna `string`es de tipo , SQL podría negarse a crear la tabla debido a otras limitaciones en la columna de clave principal. La solución consiste en crear manualmente la tabla en SQL antes de exportar los datos. La razón de esta limitación es que las columnas de clave principal en SQL no pueden ser de tamaño ilimitado, pero las columnas de tabla Kusto no tienen limitaciones de tamaño declaradas.

## <a name="azure-db-aad-integrated-authentication-documentation"></a>Documentación de autenticación integrada de AAD de Azure DB

* [Usar la autenticación de Azure Active Directory para la autenticación con SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)
* [Extensiones de autenticación de Azure AD para herramientas de Azure SQL DB y SQL DW] (https://azure.microsoft.com/blog/azure-ad-authentication-extensions-for-azure-sql-db-and-sql-dw-tools/)

**Ejemplos** 

En este ejemplo, Kusto ejecuta la consulta y, a continuación, `MySqlTable` exporta `MyDatabase` el primer `myserver`conjunto de registros generado por la consulta a la tabla de la base de datos en el servidor.

```kusto 
.export async to sql MySqlTable
    h@"Server=tcp:myserver.database.windows.net,1433;Database=MyDatabase;Authentication=Active Directory Integrated;Connection Timeout=30;"
    <| print Id="d3b68d12-cbd3-428b-807f-2c740f561989", Name="YSO4", DateOfBirth=datetime(2017-10-15)
```

En este ejemplo, Kusto ejecuta la consulta y, a continuación, `MySqlTable` exporta `MyDatabase` el primer `myserver`conjunto de registros generado por la consulta a la tabla de la base de datos en el servidor.
Si la tabla de destino no existe en la base de datos de destino, se crea.

```kusto 
.export async to sql ['dbo.MySqlTable']
    h@"Server=tcp:myserver.database.windows.net,1433;Database=MyDatabase;Authentication=Active Directory Integrated;Connection Timeout=30;"
    with (createifnotexists="true", primarykey="Id")
    <| print Message = "Hello World!", Timestamp = now(), Id=12345678
```
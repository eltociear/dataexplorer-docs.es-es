---
title: Clientes de MS-TDS y Kusto - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describen los clientes de MS-TDS y Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 233eec65c14d76b25b76cc85c7ddd190e5171dfe
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523501"
---
# <a name="ms-tds-clients-and-kusto"></a>Clientes de MS-TDS y Kusto

Kusto implementa el punto de conexión de cumplimiento de TDS para clientes MS-SQL. Dado que la compatibilidad está en el nivel de protocolo, cualquier biblioteca o aplicación que se pueda conectar a la base de datos de SQL Azure con la autenticación de Azure Active Directory, funcionaría con el servidor Kusto, ortogonalmente con el sistema operativo o en tiempo de ejecución. Solo tiene que usar el nombre de dominio del servidor Kusto como si fuera el servidor de SQL Azure.

En el nivel de lenguaje SQL, Kusto implementa el subconjunto de T-SQL y el subconjunto de emulación de SQL Server. Vea [problemas conocidos](./sqlknownissues.md) para algunas de las principales diferencias entre la implementación de SQL Server de T-SQL y Kusto.

## <a name="net-sql-client"></a>Cliente SQL de .NET

Kusto admite la autenticación de Azure Active Directory para clientes SQL.

Para obtener más información, consulte [.NET SQL Client (autenticación](./aad.md#net-sql-client-user) de usuario) y [.NET SQL Client (autenticación](./aad.md#net-sql-client-application) de aplicación)



## <a name="jdbc"></a>JDBC

El controlador JDBC de Microsoft se puede utilizar para conectarse a Kusto con autenticación AAD.

Haga la aplicación para utilizar una de las versiones de *mssql-jdbc* JAR y *adal4j* JAR y todas sus dependencias. Por ejemplo:

```s
mssql-jdbc-7.0.0.jre8.jar
adal4j-1.6.3.jar
accessors-smart-1.2.jar
activation-1.1.jar
asm-5.0.4.jar
commons-codec-1.11.jar
commons-lang3-3.5.jar
gson-2.8.0.jar
javax.mail-1.6.1.jar
jcip-annotations-1.0-1.jar
json-smart-2.3.jar
lang-tag-1.4.4.jar
nimbus-jose-jwt-6.5.jar
oauth2-oidc-sdk-5.64.4.jar
slf4j-api-1.7.21.jar
```

Realizar aplicación para utilizar la clase de controlador JDBC`com.microsoft.sqlserver.jdbc.SQLServerDriver`

Utilice la cadena de conexión como:

```s
jdbc:sqlserver://<cluster_name.region>.kusto.windows.net:1433;database=<database_name>;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.kusto.windows.net;loginTimeout=30;authentication=ActiveDirectoryPassword
```

Si desea utilizar el modo de autenticación integrado de AAD reemplace *ActiveDirectoryPassword* por *ActiveDirectoryIntegrated*

Para obtener más información, consulte [JDBC (autenticación de usuario)](./aad.md#jdbc-user) y [JDBC (autenticación de aplicaciones)](./aad.md#jdbc-application)

## <a name="odbc"></a>ODBC

Las aplicaciones que admiten ODBC pueden conectarse a Kusto.

Crear origen de datos ODBC:
1. Inicie el Administrador de orígenes de datos ODBC.
2. Crear nuevo origen `Add`de datos (botón ).
3. Seleccione `ODBC Driver 17 for SQL Server`.
3. Asigne un nombre al origen de datos y `Server` especifique el nombre del clúster de Kusto en el campo, por ejemplo.`mykusto.kusto.windows.net`
4. Seleccione `Active Directory Integrated` la opción de autenticación.
5. La siguiente pestaña permite seleccionar la base de datos.
7. Puede dejar los valores predeterminados para todos los demás ajustes en las siguientes pestañas.
8. `Finish`botón abre el cuadro de diálogo de resumen del origen de datos donde se puede probar la conexión.
9. El origen ODBC ahora se puede usar con aplicaciones.

Si la aplicación ODBC puede aceptar la cadena de conexión en su lugar o además de DSN, utilice la siguiente cadena de conexión:
```s
"Driver={ODBC Driver 17 for SQL Server};Server=mykustocluster.kusto.windows.net;Database=mykustodatabase;Authentication=ActiveDirectoryIntegrated"
```

Algunas aplicaciones ODBC no funcionan bien con el https://docs.microsoft.com/sql/relational-databases/native-client/features/using-large-value-types?view=sql-server-2017#sql-server-native-client-odbc-driver tipo NVARCHAR(MAX) (consulte para obtener más detalles). La solución común utilizada para dicha aplicación es convertir los datos devueltos a NVARCHAR(n), con algún valor para n, por ejemplo, NVARCHAR(4000). Esta solución alternativa no funcionaría para Kusto, ya que Kusto solo tiene un tipo de cadena y para los clientes SQL se codifica como NVARCHAR(MAX). Kusto ofrece diferentes soluciones para este tipo de aplicaciones. Es posible configurar Kusto para codificar todas las cadenas como NVARCHAR(n) a través de la cadena de conexión. El campo de idioma de la cadena de conexión `language@OptionName1:OptionValue1,OptionName2:OptionValue2`se puede utilizar para especificar opciones de ajuste en un formato: . 

Por ejemplo, la siguiente cadena de conexión indicará a Kusto que codifique cadenas como NVARCHAR(8000):
```s
"Driver={ODBC Driver 17 for SQL Server};Server=mykustocluster.kusto.windows.net;Database=mykustodatabase;Authentication=ActiveDirectoryIntegrated,Language=any@MaxStringSize:8000"
```

### <a name="powershell"></a>PowerShell

Ejemplo de script de PowerShell que usa el controlador ODBC:

```powershell
$conn = [System.Data.Common.DbProviderFactories]::GetFactory("System.Data.Odbc").CreateConnection()
$conn.ConnectionString = "Driver={ODBC Driver 17 for SQL Server};Server=mykustocluster.kusto.windows.net;Database=mykustodatabase;Authentication=ActiveDirectoryIntegrated"
$conn.Open()
$conn.GetSchema("Tables")
$conn.Close()  

```



## <a name="linqpad"></a>LINQPad

Es posible utilizar aplicaciones Linq con Kusto conectándose a Kusto como si fuera SQL Server.
LINQPad se puede usar para explorar la compatibilidad con Linq. También se puede utilizar para examinar Kusto y ejecutar consultas SQL.
LINQPad es la herramienta recomendada para explorar el punto de conexión de Kusto TDS (SQL).

Conéctese como si se conectara a Microsoft SQL Server. Observe que LINQPad admite la autenticación de Active Directory.

1. Haga clic en `Add connection`.
2. Elija `Build data context automatically`.
3. Elija controlador `Default (LINQ to SQL)`LINQPad .
4. Como proveedor `SQL Azure`elegir .
5. Para el servidor, especifique el nombre del clúster de Kusto, por ejemplo.`mykusto.kusto.windows.net`
6. Para login `Windows Authentication (Active Directory)`puede elegir .
7. Haga `Test` clic en el botón para verificar la conectividad.
8. Haga `OK` clic en el botón. La ventana del explorador muestra la vista de árbol con bases de datos.
9. Puede examinar bases de datos, tablas y columnas.
10. En la ventana de consulta, puede ejecutar consultas SQL. Especifique el idioma SQL y elija la conexión a la base de datos.
11. En la ventana de consulta también puede ejecutar consultas LINQ. Por ejemplo, haga clic con el botón derecho en una tabla en la ventana del navegador. Opción `Count` de selección. Déjalo correr.

## <a name="azure-data-studio-134-and-above"></a>Azure Data Studio (1.3.4 y superior)

1. Nueva conexión.
2. Seleccione el `Microsoft SQL Server`tipo de conexión: .
3. Especifique el nombre del clúster de Kusto como nombre de servidor, por ejemplo.`mykusto.kusto.windows.net`
4. Seleccione el `Azure Active Directory - Universal with MFA support`tipo de autenticación: .
5. Especifique la cuenta aprovisionada en `myname@contoso.com` AAD, por ejemplo (Agregar cuenta la primera vez).
6. Selector de base de datos se puede utilizar para seleccionar la base de datos.
7. `Connect`botón le llevaría al panel de la base de datos.
8. Haga clic con `New Query` el botón derecho `New Query` en la conexión y seleccione para abrir la pestaña de consulta o haga clic en la tarea en el panel.

## <a name="power-bi-desktop"></a>Power BI Desktop

Conéctese como se conecta a SQL Azure Database.

1. En `Get Data` `More`elegir `Azure` , luego y luego`Azure SQL Database`
2. Especifique el nombre del servidor Kusto, por ejemplo.`mykusto.kusto.windows.net`
3. Utilice la opción "DirectQuery".
4. Elija `Microsoft account` la `Windows`autenticación (no) y haga clic. `sign in`
5. El selector muestra las bases de datos disponibles. Continúe como lo haría con el servidor SQL real.

## <a name="excel"></a>Excel

Conéctese como se conecta a SQL Azure Database.

1. En `Data` `Get Data`pestaña, `From Azure`, ,`From Azure SQL Database`
2. Especifique el nombre del servidor Kusto, por ejemplo.`mykusto.kusto.windows.net`
3. Elija `Microsoft account` la `Windows`autenticación (no) y haga clic. `sign in`
4. Una vez que `Connect`haya iniciado sesión, haga clic en .
5. El selector muestra las bases de datos disponibles. Continúe como lo haría con el servidor SQL real.

## <a name="tableau"></a>Tableau

1. Crear origen de datos ODBC, consulte la sección [ODBC.](./clients.md#odbc)
2. Conéctese `Other Databases (ODBC)`a través de .
3. Seleccione Origen de `DSN`datos ODBC en .
4. Utilice `Connect` el botón para establecer la conexión.
5. Una `Sign In` vez que el botón esté disponible, inicie sesión en Kusto.

## <a name="dbeaver-533-and-above"></a>DBeaver (5.3.3 y superior)

Configure DBeaver para el manejo de conjuntos de resultados de una manera compatible con Kusto:

1. En `Window` el `Preferences`menú seleccione .
2. En `Editors` la `Data Editor`sección seleccione .
3. Asegúrese de que `Refresh data on next page reading` la opción está marcada.

Crear conexión a la base de datos Kusto:

1. Cree una nueva`Database` conexión `New Connection` de base de datos (menú y opción).
2. Busque `Azure` y `Azure SQL Database`seleccione . Presione `Next`.
3. Especifique el host, `mykusto.kusto.windows.net` por ejemplo, y `mydatabase`la base de datos, por ejemplo . No deje master como nombre de base de datos. Kusto requiere conexión a una base de datos específica.
4. Para la `Active Directory - Password`opción de autenticación, seleccione .
5. Especifique las credenciales del usuario del `myname@contoso.com` directorio activo, por ejemplo, y la contraseña correspondiente para este usuario.
6. Pulse `Test Connection …` para comprobar que los detalles de la conexión son correctos.

## <a name="microsoft-sql-server-management-studio-v18x"></a>Microsoft SQL Server Management Studio (v18.x)

1. En `Object Explorer` `Connect`, `Database Engine`, .
2. Especifique el nombre del clúster de Kusto como nombre de servidor, por ejemplo.`mykusto.kusto.windows.net`
3. Utilice `Active Directory - Integrated` la opción para la autenticación.
4. Haga clic en `Options`.
5. En `Connect to database` combo, puede navegar `Browse Server` por las bases de datos disponibles a través de la opción.
6. Haga `Yes` clic para continuar con la navegación.
7. El cuadro de diálogo muestra la vista en árbol con todas las bases de datos disponibles. Puede hacer clic en uno para continuar con la conexión a la base de datos.
8. Alternativamente, `Connect to database` en combo, `default`puede elegir . Haga clic en `Connect`. El Explorador de objetos mostraría todas las bases de datos.
9. Todavía no se admite la exploración de objetos de base de datos a través de SSMS, ya que SSMS usa subconsultas de correlación para examinar el esquema de base de datos. Kusto no admite subconsultas correlacionadas, consulte [subconsultas correlacionadas.](./sqlknownissues.md#correlated-sub-queries)
10. Haga clic en su base de datos. Haga `New Query` clic en la opción para abrir la ventana de consulta.
11. Puede ejecutar consultas SQL personalizadas desde la ventana de consulta.

## <a name="matlab-via-jdbc"></a>MATLAB (a través de JDBC)

Agregue los archivos JAR necesarios al frente de la ruta de clase estática de MATLAB. Para ello, cree un archivo *"javaclasspath.txt"* en el directorio de preferencias. Ejecute el siguiente comando en la ventana de comandos de Matlab: 

``` s
edit(fullfile(prefdir,'javaclasspath.txt'))
```

Y agregue las trayectorias completas a los archivos JAR requeridos: 

``` s
<before>
c:\full\path\to\accessors-smart-1.2.jar
c:\full\path\to\activation-1.1.jar
c:\full\path\to\adal4j-1.6.3.jar
c:\full\path\to\asm-5.0.4.jar
c:\full\path\to\commons-codec-1.11.jar
c:\full\path\to\commons-lang3-3.5.jar
c:\full\path\to\gson-2.8.0.jar
c:\full\path\to\javax.mail-1.6.1.jar
c:\full\path\to\jcip-annotations-1.0-1.jar
c:\full\path\to\json-smart-2.3.jar
c:\full\path\to\lang-tag-1.4.4.jar
c:\full\path\to\mssql-jdbc-7.0.0.jre8.jar
c:\full\path\to\nimbus-jose-jwt-6.5.jar
c:\full\path\to\oauth2-oidc-sdk-5.64.4.jar
c:\full\path\to\slf4j-api-1.7.21.jar
```

> Realmente necesita el <*antes* de> en la parte superior para agregar estos archivos al frente de la ruta de clase. Sustituya además "c:-full-path-to" con las rutas de acceso completas reales a estos archivos. 

Reinicie MATLAB para asegurarse de que estas clases están cargadas de hecho.

Antes de intentar conectar ejecute el siguiente comando (ventana de comandos de MATLAB):

``` java
java.lang.System.clearProperty('javax.xml.transform.TransformerFactory')
```

> Esto restablece *TransformerFactory* al valor predeterminado (MATLAB suele sobrecargar esto con Saxon, pero esto es incompatible con ADAL4J).

Para conectarse al punto final de Kusto TDS, envíe el siguiente comando (ventana de comandos de MATLAB):

```s
conn = database('<<KUSTO_DATABASE>>','<<AAD_USER>>','<<USER_PWD>>','com.microsoft.sqlserver.jdbc.SQLServerDriver',['jdbc:sqlserver://<<MYCLUSTER>>.kusto.windows.net:1433;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.kusto.windows.net;loginTimeout=30;authentication=ActiveDirectoryPassword;database='])
 ```

> Es correcto aquí que esto simplemente termina con *"base de datos"* y luego ningún nombre, la función "base de datos" añadirá automáticamente la primera entrada (el nombre de la base de datos) a esta cadena.
> Si desea utilizar el modo de autenticación integrado de AAD reemplace *ActiveDirectoryPassword* por *ActiveDirectoryIntegrated*

Pruebe la conexión y ejecute una consulta de ejemplo. Enviar comandos siguientes de troquel (ventana de comandos DE MATLAB):

```s
data = select(conn, 'SELECT * FROM <<KUSTO_TABLE>>')
data
```

> Sustituya *KUSTO_TABLE* por una tabla existente en Kusto



## <a name="sending-t-sql-queries-over-the-rest-api"></a>Envío de consultas T-SQL a través de la API de REST

La API REST de [Kusto](../rest/index.md) puede aceptar y ejecutar consultas T-SQL.
Para ello, envíe la solicitud al `csl` punto de conexión de consulta con la propiedad establecida en el `sql`texto de la propia consulta t-SQL y la [propiedad](../netfx/request-properties.md) `OptionQueryLanguage` request establecida en el valor .
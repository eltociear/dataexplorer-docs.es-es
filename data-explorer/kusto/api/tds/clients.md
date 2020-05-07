---
title: Clientes MS-TDS y Kusto-Azure Explorador de datos
description: En este artículo se describen los clientes MS-TDS y Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 10/30/2019
ms.openlocfilehash: b41f77fe97ce6adeeade63c00824818f4a3af721
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82862044"
---
# <a name="ms-tds-clients-and-azure-data-explorer"></a>Clientes MS-TDS y Azure Explorador de datos

Azure Explorador de datos implementa puntos de conexión compatibles con TDS para los clientes MS-SQL. La compatibilidad se encuentra en el nivel de protocolo. Cualquier biblioteca o aplicación que pueda conectarse a la base de datos de SQL Azure con autenticación de Azure Active Directory (Azure AD) funcionará con el servidor de Azure Explorador de datos. Por lo tanto, puede usar el nombre de dominio del servidor como el servidor de SQL Azure.

Azure Explorador de datos implementa un subconjunto de T-SQL y un subconjunto de la emulación de SQL Server. Para obtener más información, consulte [problemas conocidos](./sqlknownissues.md) de las diferencias entre la implementación del SQL Server de T-SQL y Azure explorador de datos.

## <a name="net-sql-client"></a>Cliente SQL de .NET

Azure Explorador de datos admite la autenticación de Azure AD para clientes SQL. Para obtener más información, vea [cliente SQL de .net (autenticación de usuario)](./aad.md#net-sql-client-user) y [cliente SQL de .net (autenticación de aplicaciones)](./aad.md#net-sql-client-application) .

## <a name="jdbc"></a>JDBC

Microsoft JDBC driver se puede usar para conectarse a Azure Explorador de datos con la autenticación de Azure AD.

Cree una aplicación para usar una de las versiones de *MSSQL-JDBC* jar y *adal4j* jar, y todas sus dependencias.
Por ejemplo,

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

Cree una aplicación para usar la clase de controlador JDBC *com. Microsoft. SqlServer. JDBC. SQLServerDriver*.

Use una cadena de conexión como la siguiente.

```s
jdbc:sqlserver://<cluster_name.region>.kusto.windows.net:1433;database=<database_name>;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.kusto.windows.net;loginTimeout=30;authentication=ActiveDirectoryPassword
```

> [!NOTE]
> Si desea usar el modo de autenticación integrada de Azure Active Directory, reemplace *ActiveDirectoryPassword* por *ActiveDirectoryIntegrated*. Para obtener más información, consulte [JDBC (autenticación de usuario)](./aad.md#jdbc-user) y [JDBC (autenticación de aplicaciones)](./aad.md#jdbc-application).

## <a name="odbc"></a>ODBC

Las aplicaciones que admiten ODBC pueden conectarse a Azure Explorador de datos.

Cree un origen de datos ODBC:

1. Inicie el administrador de orígenes de datos ODBC.
2. Seleccione **Agregar** para crear un nuevo origen de datos y establezca el *controlador ODBC 17 para SQL Server*.
3. Asigne un nombre al origen de datos y especifique el nombre del clúster de Explorador de datos de Azure en el campo **servidor** . Por ejemplo, *mykusto.kusto.Windows.net*.
4. Establezca **Active Directory integrado**para la opción de autenticación.
5. Seleccione **siguiente** para establecer la base de datos.
7. Puede dejar los valores predeterminados para todos los demás valores de las pestañas siguientes.
8. Seleccione **Finalizar** para abrir la ventana Resumen del origen de datos, donde se puede probar la conexión.

Ahora puede usar el origen de datos ODBC con las aplicaciones.

Si la aplicación ODBC puede aceptar una cadena de conexión en lugar de, o además de DSN, use lo siguiente.

```s
"Driver={ODBC Driver 17 for SQL Server};Server=mykustocluster.kusto.windows.net;Database=mykustodatabase;Authentication=ActiveDirectoryIntegrated"
```

Algunas aplicaciones ODBC no funcionan bien con el `NVARCHAR(MAX)` tipo. Para obtener más información, vea https://docs.microsoft.com/sql/relational-databases/native-client/features/using-large-value-types?view=sql-server-2017#sql-server-native-client-odbc-driver.

La solución más común consiste en convertir los datos devueltos a *nvarchar (n)*, con algún valor para n. Por ejemplo, *nvarchar (4000)*. Sin embargo, esta solución alternativa no funcionará para Azure Explorador de datos, dado que Azure Explorador de datos solo tiene un tipo de cadena y para los clientes SQL se codifica como *nvarchar (Max)*.

Azure Explorador de datos ofrece una solución alternativa diferente. Puede configurar Azure Explorador de datos para codificar todas las cadenas como *nvarchar (n)* a través de una cadena de conexión. El campo Language de la cadena de conexión se puede usar para especificar las opciones de optimización en el formato, * language@OptionName1:OptionValue1, OptionName2: OptionValue2*.

Por ejemplo, la siguiente cadena de conexión indicará a Azure Explorador de datos que codifique las cadenas como *nvarchar (8000)*.

```s
"Driver={ODBC Driver 17 for SQL Server};Server=mykustocluster.kusto.windows.net;Database=mykustodatabase;Authentication=ActiveDirectoryIntegrated,Language=any@MaxStringSize:8000"
```

### <a name="powershell"></a>PowerShell

Este es un ejemplo del script de PowerShell que usa el controlador ODBC.

```powershell
$conn = [System.Data.Common.DbProviderFactories]::GetFactory("System.Data.Odbc").CreateConnection()
$conn.ConnectionString = "Driver={ODBC Driver 17 for SQL Server};Server=mykustocluster.kusto.windows.net;Database=mykustodatabase;Authentication=ActiveDirectoryIntegrated"
$conn.Open()
$conn.GetSchema("Tables")
$conn.Close()
```

## <a name="linqpad"></a>LINQPad

Se puede usar una aplicación LINQ con Azure Explorador de datos, conectándola como es un servidor SQL Server.
Use LINQPad para explorar la compatibilidad de LINQ y examinar Azure Explorador de datos. También puede ejecutar consultas SQL y es la herramienta recomendada para explorar los puntos de conexión de Azure Explorador de datos TDS (SQL).

Conéctese como lo hace, en el Microsoft SQL Server. LINQPad admite la autenticación Active Directory.

1. Seleccione **Agregar conexión**.
2. Establezca el **contexto de datos de compilación automáticamente**.
3. Establezca el **valor predeterminado (LINQ to SQL)**, el controlador LINQPad.
4. Establezca **SQL Azure**.
5. En el servidor, especifique el nombre del clúster de Azure Explorador de datos. Por ejemplo, *mykusto.kusto.Windows.net*.
6. Establezca **autenticación de Windows (Active Directory)** para iniciar sesión.
7. Seleccione **prueba** para comprobar la conectividad.
8. Seleccione **Aceptar**. La ventana del explorador muestra la vista de árbol con las bases de datos.
9. Puede examinar las bases de datos, las tablas y las columnas.
10. Puede ejecutar consultas SQL en la ventana de consulta. Especifique el lenguaje SQL y seleccione una conexión a la base de datos.
11. También puede ejecutar consultas LINQ en la ventana de consulta. Por ejemplo, seleccione una tabla en la ventana del explorador. Seleccione **recuento**y deje que se ejecute.

## <a name="azure-data-studio-134-and-above"></a>Azure Data Studio (1.3.4 y versiones posteriores)

Cree una nueva conexión.

1. Establezca el tipo de conexión en **Microsoft SQL Server**.
2. Especifique el nombre del clúster de Azure Explorador de datos como un nombre de servidor. Por ejemplo, *mykusto.kusto.Windows.net*.
3. Establezca el tipo **de autenticación Azure Active Directory-universal con compatibilidad con MFA**.
4. Especifique la cuenta que se ha aprovisionado en el Azure AD. Por ejemplo, *myname@contoso.com* . Agregue la cuenta la primera vez.
5. Use el selector de base de datos para seleccionar la base de datos.
6. Seleccione **conectar** para pasar al panel base de datos y establecer la conexión.
7. Seleccione **nueva consulta** para abrir la ventana de consulta o seleccione la **nueva tarea consulta** en el panel.

## <a name="power-bi-desktop"></a>Power BI Desktop

Conéctese como lo hace, a la base de datos de SQL Azure.

1. En **obtener datos**, seleccione **más**.
2. Establezca **Azure**y **Azure SQL Database**.
3. Especifique el nombre del servidor de Azure Explorador de datos. Por ejemplo, *mykusto.kusto.Windows.net*.
4. Seleccione **DirectQuery**.
5. Establezca autenticación **cuenta de Microsoft** (no **Windows**) y seleccione **iniciar sesión**.
6. El selector de base de datos muestra las bases de datos disponibles. Continúe como lo hace, con un servidor SQL Server real.

## <a name="excel"></a>Excel

Conéctese como lo hace, a la base de datos de SQL Azure.

1. Seleccione **obtener datos** en la pestaña **datos** y, a continuación, establezca **desde Azure** y **desde Azure SQL Database**.
2. Especifique el nombre del servidor de Azure Explorador de datos. Por ejemplo, *mykusto.kusto.Windows.net*.
3. Establezca autenticación **cuenta de Microsoft** (no **Windows**) y seleccione **iniciar sesión**.
5. El selector de base de datos muestra las bases de datos disponibles. Continúe como lo hace, con un servidor SQL Server real.
4. Una vez que haya iniciado sesión, seleccione **conectar**.
5. El selector de base de datos muestra las bases de datos disponibles. Continúe como lo hace, con un servidor SQL Server real.

## <a name="tableau"></a>Tableau

Cree un origen de datos ODBC. Para obtener más información, vea la sección [ODBC](./clients.md#odbc) .

1. Conectarse a través de **otras bases de datos (ODBC)**.
2. Establezca el origen de datos ODBC en **DSN**.
3. Seleccione **conectar** para establecer una conexión.
4. Seleccione **iniciar sesión**, una vez que el botón esté disponible e inicie sesión en Azure explorador de datos.

## <a name="dbeaver-533-and-above"></a>DBeaver (5.3.3 y versiones posteriores)

Configure DBeaver para administrar los conjuntos de resultados de una manera compatible con Azure Explorador de datos.

1. Seleccione **preferencias** en el menú **ventana** .
2. Seleccione **Editor de datos** en la sección **editores** .
3. Asegúrese de que la **actualización de datos en la siguiente lectura de página** esté marcada.

Cree una conexión a la base de datos de Azure Explorador de datos.

1. Seleccione **nueva conexión** en el menú **base de datos** .
2. Busque **Azure** y establezca **Azure SQL Database**. Seleccione **Next** (Siguiente).
3. Especifique el host. Por ejemplo, *mykusto.kusto.Windows.net*.
4. Especifique la base de datos. Por ejemplo, *base de datos*.

> [!WARNING]
> No utilice *Master* como nombre de la base de datos. Azure Explorador de datos requiere una conexión a una base de datos específica.

5. Establezca **Active Directory: contraseña para la** *autenticación*.
6. Especifique las credenciales del usuario de Active Directory. Por ejemplo, *myname@contoso.com*y establezca la contraseña correspondiente para este usuario.
7. Seleccione **probar conexión..** . para comprobar que los detalles de conexión son correctos.

## <a name="microsoft-sql-server-management-studio-v18x"></a>Microsoft SQL Server Management Studio (V18. x)

1. Seleccione **conectar**y **motor de base de datos** en **Explorador de objetos**.
2. Especifique el nombre del clúster de Azure Explorador de datos como un nombre de servidor. Por ejemplo, *mykusto.kusto.Windows.net*.
3. Establezca **Active Directory integrado para la** autenticación.
4. Seleccione **Opciones**.
5. Seleccione **examinar servidor** en **conectar con base de datos** para examinar las bases de datos disponibles.
6. Seleccione **sí** para continuar con la exploración.
7. La ventana muestra una vista de árbol con todas las bases de datos disponibles. Seleccione una para conectarse a la base de datos.
8. Otra posibilidad consiste en seleccionar **predeterminado** en **conectar con base de datos**y, a continuación, seleccionar **conectar**. El explorador de objetos mostrará todas las bases de datos.

> [!NOTE]
> Todavía no se admite la exploración de objetos de base de datos a través de SSMS, ya que SSMS usa subconsultas correlacionate para examinar el esquema de la base de datos.
> Las subconsultas correlacionadas no son compatibles con Azure Explorador de datos. Para obtener más información, consulte [subconsultas correlacionadas](./sqlknownissues.md#correlated-sub-queries).

10. Seleccione **nueva consulta** para abrir la ventana de consulta y establecer la base de datos.

Puede ejecutar consultas SQL personalizadas desde la ventana de consulta.

## <a name="matlab-via-jdbc"></a>MATLAB (a través de JDBC)

Agregue los archivos JAR necesarios al principio de la ruta de clases estática de MATLAB mediante la creación de un archivo *"javaclasspath. txt"* en el directorio de preferencias.

1. Ejecute el siguiente comando en la ventana de comandos de MATLAB.

``` s
edit(fullfile(prefdir,'javaclasspath.txt'))
```

2. Agregue las rutas de acceso completas a los archivos JAR necesarios.

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

> [!NOTE]
> Necesita el <**antes** de> en la parte superior, de modo que estos archivos se agregan al principio de la ruta de clases. Además, reemplace *c:\full\path\to* por las rutas de acceso completas reales a estos archivos.

3. Reinicie MATLAB para asegurarse de que se cargan estas clases.

4. Antes de intentar conectarse, ejecute el siguiente comando (ventana de comandos de MATLAB).

``` java
java.lang.System.clearProperty('javax.xml.transform.TransformerFactory')
```

> [!NOTE]
> Esto restablece el valor predeterminado de **TransformerFactory** (MATLAB normalmente sobrecarga esto con **Saxon**, pero esto es incompatible con **ADAL4J**).

5. Conéctese al punto de conexión TDS de Azure Explorador de datos con el siguiente comando (ventana de comandos de MATLAB).

```s
conn = database('<<KUSTO_DATABASE>>','<<AAD_USER>>','<<USER_PWD>>','com.microsoft.sqlserver.jdbc.SQLServerDriver',['jdbc:sqlserver://<<MYCLUSTER>>.kusto.windows.net:1433;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.kusto.windows.net;loginTimeout=30;authentication=ActiveDirectoryPassword;database='])
 ```

> [!NOTE]
> Es correcto terminar con **"Database ="** y, a continuación, sin valor. La función de *base de datos* anexará automáticamente la primera entrada, el nombre de la base de datos, a esta cadena.
> Si desea usar Azure Active Directory modo de autenticación integrada, reemplace *ActiveDirectoryPassword* por *ActiveDirectoryIntegrated*.

6. Pruebe la conexión y ejecute una consulta de ejemplo. Envíe los siguientes comandos (ventana de comandos de MATLAB).

```s
data = select(conn, 'SELECT * FROM <<KUSTO_TABLE>>')
data
```

> [!NOTE]
> Reemplace *KUSTO_TABLE* por una tabla existente en Azure explorador de datos.

## <a name="sending-t-sql-queries-over-the-rest-api"></a>Envío de consultas de T-SQL a través de la API de REST

La [API de REST de Azure explorador de datos](../rest/index.md) puede aceptar y ejecutar consultas de T-SQL.

1. Envíe la solicitud al punto de conexión de la consulta con la propiedad de **CSL** establecida en el texto de la consulta T-SQL.
2. Establezca la **[propiedad de solicitud](../netfx/request-properties.md)** **OptionQueryLanguage** en **SQL**.

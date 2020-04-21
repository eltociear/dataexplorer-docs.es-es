---
title: MS-TDS con Azure Active Directory - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe MS-TDS con Azure Active Directory en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/02/2019
ms.openlocfilehash: e70f4e9fa4d831d3a1c2eeb60f07a959a65e478e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523518"
---
# <a name="ms-tds-with-azure-active-directory"></a>MS-TDS con Azure Active Directory

## <a name="aad-user-authentication"></a>Autenticación de usuario de AAD

Los clientes SQL que admiten la autenticación de usuario de AAD se pueden usar con Kusto.

### <a name="net-sql-client-user"></a>Cliente SQL de .NET (usuario)

Por ejemplo, para AAD integrado:
```csharp
    var csb = new SqlConnectionStringBuilder()
    {
        InitialCatalog = "mydatabase",
        Authentication = SqlAuthenticationMethod.ActiveDirectoryIntegrated,
        DataSource = "mykusto.kusto.windows.net"
    };
```

Kusto admite la autenticación con token de acceso ya obtenido:
```csharp
    var csb = new SqlConnectionStringBuilder()
    {
        InitialCatalog = "mydatabase",
        DataSource = "mykusto.kusto.windows.net"
    };
    using (var connection = new SqlConnection(csb.ToString()))
    {
        connection.AccessToken = accessToken;
        await connection.OpenAsync();
    }
```

### <a name="jdbc-user"></a>JDBC (usuario)

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.Statement;
import com.microsoft.sqlserver.jdbc.SQLServerDataSource;
import com.microsoft.aad.adal4j.*;

public class Sample {
  public static void main(String[] args) throws Exception {
    AuthenticationResult authenticationResult = futureAuthenticationResult.get();
    SQLServerDataSource ds = new SQLServerDataSource();
    ds.setServerName("<your cluster DNS name>");
    ds.setDatabaseName("<your database name>");
    ds.setHostNameInCertificate("*.kusto.windows.net"); // Or appropriate regional domain.
    ds.setAuthentication("ActiveDirectoryIntegrated");
    try (Connection connection = ds.getConnection(); 
         Statement stmt = connection.createStatement();) {
      ResultSet rs = stmt.executeQuery("<your T-SQL query>");
      /* 
      Read query result. 
      */
    } catch (Exception e) {
      System.out.println();
      e.printStackTrace();
    }
  }
}
```

## <a name="aad-application-authentication"></a>Autenticación de aplicaciones de AAD

La aplicación AAD aprovisionada para Kusto puede usar bibliotecas de cliente SQL que admiten AAD para conectarse a Kusto. Consulte Creación de una aplicación de [AAD](../../management/access-control/how-to-provision-aad-app.md) para obtener más información acerca de las aplicaciones de AAD.

### <a name="net-sql-client-application"></a>Cliente SQL de .NET (aplicación)

Suponiendo que ha aprovisionado la aplicación AAD con *ApplicationClientId* y *ApplicationKey* y le ha concedido permisos para tener acceso a *databaseName* de base de datos en el clúster *ClusterDnsName*, en el ejemplo siguiente se muestra cómo usar .NET SQL Client para consultas desde esta aplicación de AAD.

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System;
using System.Data;
using System.Data.SqlClient;

namespace Sample
{
  class Program
  {
    private static async Task<string> ObtainToken()
    {
      var authContext = new AuthenticationContext(
        // Can also use tenant ID.
        "https://login.microsoftonline.com/<your AAD tenant name>");
      var applicationCredentials = new ClientCredential(
        "<your application client ID>", 
        "<your application key>");
      var result = await authContext.AcquireTokenAsync(
        "https://<your cluster DNS name>", 
        applicationCredentials);
      return result.AccessToken;
    }

    private static async Task QuerySample()
    {
      var csb = new SqlConnectionStringBuilder()
      {
        InitialCatalog = "<your database name>",
        DataSource = "<your cluster DNS name>"
      };
      using (var connection = new SqlConnection(csb.ToString()))
      {
        connection.AccessToken = await ObtainToken();
        await connection.OpenAsync();
        using (var command = new SqlCommand(
          "<your T-SQL query>", 
          connection))
        {
          var reader = await command.ExecuteReaderAsync();
          /*
          Read query result.
          */
        }
      }
    }
  }
}
```

### <a name="jdbc-application"></a>JDBC (aplicación)

```java
import java.sql.*;
import com.microsoft.sqlserver.jdbc.*;
import com.microsoft.aad.adal4j.*;

public class Sample {
  public static void main(String[] args) throws Throwable {
    ExecutorService service = Executors.newFixedThreadPool(1);
    // Can also use tenant name.
    String url = "https://login.microsoftonline.com/<your AAD tenant ID>"; 
    AuthenticationContext authenticationContext = 
      new AuthenticationContext(url, false, service);
    ClientCredential  clientCredential = new ClientCredential(
      "<your application client ID>", 
      "<your application key>");
    Future<AuthenticationResult> futureAuthenticationResult = 
      authenticationContext.acquireToken(
        "https://<your cluster DNS name>", 
        clientCredential, 
        null);
    AuthenticationResult authenticationResult = futureAuthenticationResult.get();
    SQLServerDataSource ds = new SQLServerDataSource();
    ds.setServerName("<your cluster DNS name>");
    ds.setDatabaseName("<your database name>");
    ds.setAccessToken(authenticationResult.getAccessToken());
    connection = ds.getConnection();
    statement = connection.createStatement();
    ResultSet rs = statement.executeQuery("<your T-SQL query>");
    /*
    Read query result.
    */
  }
}
```
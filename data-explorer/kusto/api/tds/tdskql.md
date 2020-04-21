---
title: KQL sobre TDS - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe KQL sobre TDS en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/09/2019
ms.openlocfilehash: 364db99141c528f4a822692124ebb870d7315bca
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523314"
---
# <a name="kql-over-tds"></a>KQL sobre TDS

Kusto permite utilizar el punto de conexión de TDS para ejecutar consultas creados en el lenguaje de consulta [KQL](../../query/index.md) nativo. Esta funcionalidad ofrece una migiración más suave hacia Kusto. Por ejemplo, es posible crear un trabajo SSIS para consultar Kusto con la consulta KQL.

## <a name="executing-kusto-stored-functions"></a>Ejecución de funciones almacenadas de Kusto

Kusto permite ejecutar [funciones almacenadas](../../query/schema-entities/stored-functions.md) de forma similar a llamar a procedimientos almacenados SQL.

Por ejemplo, la función almacenada MyFunction:

|NOMBRE |Parámetros|Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction |(myLimit: long)| •StormEvents &#124; limitar myLimit|MyFolder|Función de demostración con parámetro||

se puede llamar así:

```csharp
  using (var connection = new SqlConnection(csb.ToString()))
  {
    await connection.OpenAsync();
    using (var command = new SqlCommand("kusto.MyFunction", connection))
    {
      command.CommandType = CommandType.StoredProcedure;
      var parameter = new SqlParameter("mylimit", SqlDbType.Int);
      command.Parameters.Add(parameter);
      parameter.Value = 3;
      using (var reader = await command.ExecuteReaderAsync())
      {
        // Read the response.
      }
    }
  }
```

Tenga en cuenta que las funciones `kusto`almacenadas deben llamarse con un esquema explícito denominado , para distinguir entre las funciones almacenadas de Kusto y los procedimientos almacenados del sistema SQL emulados.

Las funciones almacenadas de Kusto también se pueden llamar desde T-SQL, al igual que las funciones tabulares SQL:

```sql
SELECT * FROM kusto.MyFunction(10)
```

Se recomienda crear consultas KQL optimizadas y encapsularlas en funciones almacenadas, lo que hace que el código de consulta de T-SQL sea mínimo.

## <a name="executing-kql-query"></a>Ejecución de la consulta KQL

De forma `sp_executesql`similar a SQL Server, `sp_execute_kql` Kusto introdujo un procedimiento almacenado para ejecutar consultas [KQL,](../../query/index.md) incluidas las consultas parametrizadas.

El 1st `sp_execute_kql` parámetro de es la consulta KQL. Se pueden introducir parámetros adicionales y actuarán como parámetros de [consulta.](../../query/queryparametersstatement.md)

Por ejemplo:

```csharp
  using (var connection = new SqlConnection(csb.ToString()))
  {
    await connection.OpenAsync();
    using (var command = new SqlCommand("sp_execute_kql", connection))
    {
      command.CommandType = CommandType.StoredProcedure;
      var query = new SqlParameter("@kql_query", SqlDbType.NVarChar);
      command.Parameters.Add(query);
      var parameter = new SqlParameter("mylimit", SqlDbType.Int);
      command.Parameters.Add(parameter);
      query.Value = "StormEvents | limit myLimit";
      parameter.Value = 3;
      using (var reader = await command.ExecuteReaderAsync())
      {
        // Read the response.
      }
    }
  }
```

Tenga en cuenta que no hay necesidad de declarar parámetros al llamar a través de TDS, ya que los tipos de parámetros se establecen a través del protocolo.
---
title: 'KQL sobre TDS: Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describen KQL sobre TDS en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/09/2019
ms.openlocfilehash: 071658bf2277dd0ddb4734aaf0b59a7a44c8fe27
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512357"
---
# <a name="kql-over-tds"></a>KQL sobre TDS

Kusto permite a los extremos TDS ejecutar consultas creadas en el lenguaje de consulta [KQL](../../query/index.md) nativo. Esta capacidad permite una migración más fluida hacia Kusto. Por ejemplo, puede crear trabajos de SSIS para consultar Kusto con una consulta de KQL.

## <a name="executing-kusto-stored-functions"></a>Ejecutar funciones almacenadas de Kusto

Kusto permite que se ejecuten [funciones almacenadas](../../query/schema-entities/stored-functions.md) , como llamar a procedimientos almacenados de SQL.

Por ejemplo, la función almacenada myFunction:

|Name |Parámetros|Cuerpo|Carpeta|DocString
|---|---|---|---|---
|MyFunction |(malimit: Long)| {StormEvents &#124; limitar el límite}|MyFolder|Función demo con el parámetro||

se puede llamar de la siguiente manera:

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

> [!NOTE]
> Llamar a funciones almacenadas con un esquema explícito denominado `kusto` , para distinguir entre las funciones almacenadas de Kusto y los procedimientos almacenados del sistema SQL emulados.

También puede llamar a las funciones almacenadas de Kusto desde T-SQL, como las funciones tabulares de SQL:

```sql
SELECT * FROM kusto.MyFunction(10)
```

Crear consultas de KQL optimizadas y encapsularlas en funciones almacenadas para que el código de consulta T-SQL sea mínimo.

## <a name="executing-kql-query"></a>Ejecutando consulta de KQL

El procedimiento almacenado `sp_execute_kql` ejecuta consultas de [KQL](../../query/index.md) (incluidas las consultas con parámetros). Este procedimiento es similar a SQL Server `sp_executesql` .

El primer parámetro de `sp_execute_kql` es la consulta de KQL. Puede introducir parámetros adicionales y actuarán como [parámetros de consulta](../../query/queryparametersstatement.md).

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

> [!NOTE]
> No es necesario declarar parámetros al llamar a través de TDS, ya que los tipos de parámetro se establecen a través del protocolo.

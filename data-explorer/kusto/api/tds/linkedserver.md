---
title: 'Kusto como servidor vinculado de SQL Server: Azure Explorador de datos'
description: En este artículo se describe Kusto como un servidor vinculado de SQL Server en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 103d39f7fdd10f0375e4a530d51cd17f7cdcec51
ms.sourcegitcommit: 061eac135a123174c85fe1afca4d4208c044c678
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/05/2020
ms.locfileid: "82799601"
---
# <a name="kusto-as-a-linked-server-from-the-sql-server"></a>Kusto como servidor vinculado de SQL Server

SQL Server local permite asociar un servidor vinculado y permite crear consultas que combinen datos de SQL Server y de servidores vinculados.

Puede usar Kusto como servidor vinculado a través de la conectividad ODBC.
El servicio local SQL Server debe usar una cuenta de Active Directory (no la cuenta de servicio predeterminada) que le permita conectarse a Azure Explorador de datos mediante Azure Active Directory (Azure AD).

1. Instale el controlador ODBC más reciente para SQL Server 2017 (también se incluye en SSMS 18):https://www.microsoft.com/download/details.aspx?id=56567
1. Preparar la cadena de conexión sin DSN para el controlador ODBC, para un clúster y una base de datos específicos `Driver={ODBC Driver 17 for SQL Server};Server=<cluster>.kusto.windows.net;Database=<database>;Authentication=ActiveDirectoryIntegrated;Language=any@MaxStringSize:4000`de Azure explorador de datos:. Se ha agregado la opción Language para optimizar Azure Explorador de datos para codificar cadenas como NVARCHAR (4000). Para obtener más información sobre esta solución, vea [ODBC](./clients.md#odbc).
1. Cree el servidor vinculado con la configuración a la que apuntan las flechas rojas.

:::image type="content" source="../images/linkedserverconnection.png" alt-text="conexión de servidor vinculado":::

1. Defina la seguridad con la configuración a la que apunta la flecha roja. 

:::image type="content" source="../images/linkedserverlogin.png" alt-text="Inicio de sesión de servidor vinculado":::

Para consultar datos de Kusto:

```sql
SELECT * FROM OpenQuery(LINKEDSERVER, 'SELECT * FROM <KustoStoredFunction>[(<Parameters>)]')
```

> [!NOTE]
> 1. Use [funciones almacenadas](../../query/schema-entities/stored-functions.md) de Kusto para extraer datos de Azure explorador de datos. La función almacenada puede incluir toda la lógica necesaria para las consultas eficientes de Kusto, creadas en el lenguaje [KQL](../../query/index.md) nativo y controladas por los valores de parámetro especificados. La sintaxis de T-SQL para llamar a la función almacenada Kusto es idéntica a llamar a la función tabular de SQL.
> 1. SQL Server no admite la llamada a funciones tabulares remotas desde servidores vinculados dentro de sus propias consultas. La solución alternativa para esta limitación es usar `OpenQuery` para ejecutar consultas remotas en el servidor vinculado. De esta manera, se llama a la función tabular no en el directorio de SQL Server, sino en una consulta que se ejecuta en el servidor vinculado. La consulta T-SQL externa se puede usar para combinar entre los datos de SQL Server y los datos devueltos por la función `OpenQuery`almacenada Kusto a través de.

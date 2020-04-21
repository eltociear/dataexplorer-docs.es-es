---
title: Kusto como servidor vinculado desde SQL Server - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Kusto como servidor vinculado desde SQL Server en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 77db975f2bd7c5c1d747902248070664cd020e39
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523467"
---
# <a name="kusto-as-linked-server-from-sql-server"></a>Kusto como servidor vinculado desde SQL Server

SQL Server local permite adjuntar el servidor vinculado. Esta característica permite crear consultas que unen datos desde SQL Server y desde servidores vinculados.

Es posible utilizar Kusto como servidor vinculado a través de la conectividad ODBC.

1. El servicio SQL Server (local) debe usar una cuenta de directorio activo (no la cuenta de servicio predeterminada) que permita conectarse a Kusto mediante AAD.
2. Instale el controlador ODBC más reciente para SQL Server 2017 (también viene con SSMS 18):https://www.microsoft.com/download/details.aspx?id=56567
3. Prepare la cadena de conexión DSN menos para el controlador `Driver={ODBC Driver 17 for SQL Server};Server=<cluster>.kusto.windows.net;Database=<database>;Authentication=ActiveDirectoryIntegrated;Language=any@MaxStringSize:4000`ODBC para un clúster y una base de datos de Kusto específicos: . La opción de idioma se agrega para ajustar Kusto para codificar cadenas como NVARCHAR(4000). Consulte más detalles sobre esta solución alternativa en [ODBC](./clients.md#odbc).
4. Cree el servidor vinculado con los siguientes 3 ajustes definidos: ![texto alternativo](../images/linkedserverconnection.png "conexión de servidor vinculado")
5. La pestaña Seguridad debe definirse con esta configuración: ![texto alternativo](../images/linkedserverlogin.png "inicio de sesión del servidor vinculado")

Ahora, es posible consultar datos de Kusto, como este:
```sql
SELECT * FROM OpenQuery(LINKEDSERVER, 'SELECT * FROM <KustoStoredFunction>[(<Parameters>)]')
```

Notas:
1. Se recomienda utilizar [funciones almacenadas](../../query/schema-entities/stored-functions.md) de Kusto para extraer datos de Kusto. La función almacenada puede incluir toda la lógica necesaria para una consulta eficaz desde Kusto, creado en lenguaje [KQL](../../query/index.md) nativo y controlado por valores de parámetro especificados. La sintaxis de T-SQL para llamar a la función almacenada de Kusto es idéntica a llamar a la función tabular SQL.
2. SQL ServerSQL Server no admite la llamada a funciones tabulares remotas desde servidores vinculados dentro de sus propias consultas. La solución alternativa para `OpenQuery` esta limitación es usar para ejecutar consultas remotas en el servidor vinculado. De esta manera, se llama a la función tabular no en la directidad de SQL Server, sino en la consulta que se ejecuta en el servidor vinculado. La consulta T-SQL externa se puede utilizar para unir entre datos en el `OpenQuery`servidor SQL y datos devueltos desde la función almacenada de Kusto a través de .
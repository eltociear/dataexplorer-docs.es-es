---
title: 'MS-TDS (compatibilidad con T-SQL): Azure Data Explorer | Microsoft Docs'
description: En este artículo se describe MS-TDS (compatibilidad con T-SQL) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/06/2019
ms.openlocfilehash: 8aaea26b6c4e8a7f76c4129faeb681791f25ae7e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490555"
---
# <a name="ms-tds-t-sql-support"></a>MS-TDS (compatibilidad con T-SQL)

Kusto admite un subconjunto del protocolo de comunicación de Microsoft SQL Server (MS-TDS), con un subconjunto del lenguaje de consulta T-SQL, para que las herramientas existentes que saben cómo consultar a SQL Server puedan trabajar con Kusto. Entre los clientes admitidos se encuentran Microsoft Excel, Microsoft Power BI y muchos otros.

Tenga en cuenta que, para que una herramienta de cliente consulte Kusto mediante MS-TDS, el cliente debe usar la autenticación integrada de Azure Active Directory.

Consulte [T-SQL](./t-sql.md) para más información acerca del lenguaje de consulta T-SQL tal y como lo implementa Kusto. 

Consulte [Clientes de MS-TDS y Kusto](./clients.md) para ver ejemplos de cómo usar Kusto desde algunos clientes conocidos mediante MS-TDS/T-SQL.

En [Kusto como servidor vinculado a SQL Server](./linkedserver.md), puede ver cómo configurar un clúster de Kusto como servidor vinculado a SQL Server en un entorno local.

Consulte [MS-TDS con Azure Active Directory](./aad.md) para más información acerca del uso de AAD mediante TDS para conectarse a Kusto.

Consulte [KQL sobre TDS](./tdskql.md) para más información acerca de cómo ejecutar consultas de KQL nativas mediante un punto de conexión de TDS. 

Por último, en [este artículo](./sqlknownissues.md) encontrará algunas de las principales diferencias entre la implementación en SQL Server de T-SQL y Kusto.
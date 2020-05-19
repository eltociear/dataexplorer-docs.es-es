---
title: 'Compatibilidad con T-SQL de MS-TDS: Azure Data Explorer'
description: En este artículo se presenta la compatibilidad con T-SQL de MS-TDS en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/06/2019
ms.openlocfilehash: a128db995c78c0583bc7c7712c06292a2f6598d1
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550544"
---
# <a name="ms-tds-t-sql-support"></a>Compatibilidad con T-SQL de MS-TDS

Azure Data Explorer (Kusto) admite un subconjunto del protocolo de comunicación de Microsoft SQL Server (MS-TDS), con un subconjunto del lenguaje de consulta T-SQL. Microsoft Excel y Microsoft Power BI son solo algunas de las muchas herramientas que pueden funcionar con Azure Data Explorer (Kusto). Estas aplicaciones de Microsoft también saben consultar SQL Server.

> [!NOTE]
> Use la autenticación integrada de Azure Active Directory (Azure AD) como herramienta de cliente para consultar Kusto a través de MS-TDS.

## <a name="next-steps"></a>Pasos siguientes

* [T-SQL](./t-sql.md): información sobre el lenguaje de consulta T-SQL tal como lo implementa Kusto. 

* [KQL sobre TDS](./tdskql.md): aprenda a ejecutar consultas de KQL nativas mediante puntos de conexión de TDS.

* [Clientes de MS-TDS y Kusto](./clients.md): use Azure Data Explorer desde clientes conocidos que utilicen MS-TDS/T-SQL.

* [Azure Data Explorer (Kusto) como servidor vinculado a SQL Server](./linkedserver.md): configure el clúster como servidor vinculado a SQL Server en un entorno local. 

* [MS-TDS con Azure Active Directory](./aad.md): use Azure AD a través de TDS para conectarse a Azure Data Explorer.

* [Problemas conocidos de SQL](./sqlknownissues.md): obtenga información sobre las principales diferencias entre la implementación en SQL Server de T-SQL y Azure Data Explorer.

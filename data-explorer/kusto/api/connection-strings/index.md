---
title: 'Cadenas de conexión: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describen las cadenas de conexión de Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 53c3caecc373a646162016fc1717ce1b0b0b85d1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490079"
---
# <a name="connection-strings"></a>Cadenas de conexión

Las cadenas de conexión se usan ampliamente en los comandos de control de Kusto, en la API de Kusto y, en ocasiones, también en consultas.
Las cadenas de conexión proporcionan un medio general para describir cómo buscar e interactuar con recursos externos a Kusto, como los blobs del servicio de Azure Blob Storage y las bases de datos de Azure SQL Database.

Hay varios formatos de las cadenas de conexión:

* Las [cadenas de conexión de Kusto](./kusto.md) describen cómo comunicarse con un punto de conexión de servicio de Kusto.
  Las cadenas de conexión de Kusto se modelan siguiendo el [modelo de cadenas de conexión de ADO.NET](https://docs.microsoft.com/dotnet/framework/data/adonet/connection-string-syntax).
* Las [cadenas de conexión de Storage](./storage.md) describen cómo apuntar Kusto a en un servicio de almacenamiento externo, como Azure Blob Storage y Azure Data Lake Storage.
* Las cadenas de conexión de SQL (las usa el [complemento sql_request](../../query/sqlrequestplugin.md) de Kusto para emitir solicitudes al servicio Azure DB y el comando [Export to SQL ](../../management/data-export/export-data-to-sql.md)).  
  Estas cadenas de conexión se ajustan a la especificación [de las cadenas de conexión SqlClient](https://docs.microsoft.com/dotnet/framework/data/adonet/connection-string-syntax#sqlclient-connection-strings).
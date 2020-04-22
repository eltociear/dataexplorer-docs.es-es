---
title: 'Introducción a Azure Data Explorer API: Azure Data Explorer | Microsoft Docs'
description: En este artículo se proporciona información general acerca de Azure Data Explorer API en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: c791ae0be0c895a4744e761c58e7d455aa0a22b9
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610277"
---
# <a name="azure-data-explorer-api-overview"></a>Introducción a Azure Data Explorer API

El servicio Azure Data Explorer admite los siguientes puntos de conexión de comunicación:

1. Un punto de conexión de [API REST](#rest-api), a través del que es posible consultar y administrar los datos en Azure Data Explorer.
   Este punto de conexión admite el [lenguaje de consulta Kusto](../query/index.md) para las consultas y los [comandos de control](../management/index.md).
2. Un punto de conexión de [MS-TDS](#ms-tds), que implementa un subconjunto del protocolo Microsoft Tabular Data Stream (TDS), que utilizan los productos de Microsoft SQL Server.
   Este punto de conexión es útil principalmente para las herramientas existentes que saben cómo comunicarse con un punto de conexión de SQL Server para las consultas.
3. Un punto de conexión de [Azure Resource Manager (ARM)](https://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftkusto), que es el medio estándar para que los servicios de Azure administren recursos como los clústeres de Azure Data Explorer.

Azure Data Explorer proporciona varias bibliotecas cliente que hacen uso de los puntos de conexión anteriores para facilitar el acceso mediante programación:

1. .NET SDK
2. SDK de Python
3. SDK de Java
4. SDK de Node
5. Go SDK
6. PowerShell
7. R

## <a name="rest-api"></a>API DE REST

El medio de comunicación principal de cualquier servicio de Azure Data Explorer es la API REST del servicio. A través de este punto de conexión completamente documentado, los autores de las llamadas pueden:

1. Consultar datos
2. Consultar y modificar metadatos
3. Ingerir datos
4. Consultar el estado del mantenimiento del servicio
5. Administrar recursos

Los diferentes servicios de Azure Data Explorer se comunican entre ellos mediante la misma API REST disponible públicamente.

Además de admitir que se realicen solicitudes a Azure Data Explorer mediante la API RES, hay varias bibliotecas cliente disponibles para usar el servicio sin tener que utilizar el protocolo de la API REST.

## <a name="ms-tds"></a>MS-TDS

Como medio alternativo para conectarse a Azure Data Explorer y consultar sus datos, Azure Data Explorer admite el protocolo de comunicación de Microsoft SQL Server (MS-TDS) e incluye una compatibilidad limitada para ejecutar consultas de T-SQL. Esto permite a los usuarios ejecutar consultas en Azure Data Explorer mediante una sintaxis de consulta conocida (T-SQL) y sus conocidas herramientas de cliente de base de datos (como LINQPad, sqlcmd, Tableau, Excel y Power BI)

En [esta página](tds/index.md) puede encontrar más detalles acerca de MS-TDS.

## <a name="net-framework-libraries"></a>Bibliotecas de .NET Framework

El uso de bibliotecas de .NET Framework es la forma recomendada de invocar la funcionalidad de Azure Data Explorer mediante programación.
Se proporcionan varias bibliotecas:

- [**Kusto. Data (biblioteca cliente de Kusto)** ](./netfx/about-kusto-data.md), que se puede usar para consultar datos, consultar metadatos y modificarlos.
- [**Kusto.Ingest (biblioteca de ingesta de Kusto)** ](netfx/about-kusto-ingest.md), que usa Kusto.Data y lo amplía para facilitar la ingesta de datos.


La **biblioteca cliente de Kusto** (Kusto.Data) se basa en la API REST de Kusto y envía las solicitudes HTTPS al clúster de Kusto de destino. 

La **biblioteca de ingesta de Kusto** (Kusto.Ingest) utiliza Kusto.Data.



Todas las bibliotecas anteriores usan las API de Azure (por ejemplo, Azure Storage API o Azure Active Directory API).

## <a name="python-libraries"></a>Bibliotecas de Python

Azure Data Explorer proporciona una biblioteca cliente de Python que permite a los autores de llamadas enviar consultas de datos y comandos de control.

## <a name="r-library"></a>Biblioteca de R

Azure Data Explorer proporciona una biblioteca cliente de R que permite a los autores de llamadas enviar consultas de datos y comandos de control.



## <a name="using-azure-data-explorer-from-powershell"></a>Uso de Azure Data Explorer desde PowerShell

Los scripts de PowerShell pueden usar las bibliotecas .NET Framework de Azure Data Explorer.
En [Llamar a Azure Data Explorer desde PowerShell](powershell/powershell.md) encontrará un ejemplo.

## <a name="ide-integration"></a>Integración del IDE

El paquete `monaco-kusto` admite la integración con Monaco Editor, que es un editor web desarrollado por Microsoft y la base de Visual Studio Code.
Obtenga más información sobre el paquete [monaco-kusto](monaco/monaco-kusto.md).
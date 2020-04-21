---
title: 'Información general sobre las bibliotecas de clientes: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe información general sobre las bibliotecas de cliente en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 2eb567062402f3c00fce6b12bf81feb507ca6c23
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610169"
---
# <a name="client-libraries-overview"></a>Descripción general de las bibliotecas de clientes

En la tabla siguiente se enumeran las diferentes bibliotecas proporcionadas para la consulta, la ingesta y la administración de ARM/RP. El uso de estas bibliotecas es la forma recomendada de hacer uso de las API de Azure e invocar la funcionalidad de Azure Data Explorer mediante programación. 


|    Idioma-Funcionalidad        |    Consultar        |    Ingesta de datos        |    Gestión ARM/RP        |
|------------------------------    |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------    |--------------------------------------------------------------------------------------------------------------------------------------------------------------------    |------------------------------------------------------------------------------------------------------------------------------    |
|    .Net        |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Data/)            |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Ingest/)        |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/1.0.0)         |
|    Estándar .Net        |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Data.NETStandard/)        |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Ingest.NETStandard/)        |            |
|    Java        |    [Maven](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-data) [GitHub](https://github.com/Azure/azure-kusto-java/tree/master/data)        |    [Maven](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-ingest) [GitHub](https://github.com/Azure/azure-kusto-java/tree/master/ingest)        |    [Maven](https://mvnrepository.com/artifact/com.microsoft.azure.kusto.v2019_01_21/azure-mgmt-kusto)        |
|    JavaScript        |             |             |    [Npm](https://www.npmjs.com/package/@azure/arm-kusto)         |
|    NodeJS        |    [npm](https://www.npmjs.com/package/azure-kusto-data) [GitHub](https://github.com/Azure/azure-kusto-node/tree/master/azure-kusto-data)        |    [npm](https://www.npmjs.com/package/azure-kusto-ingest)       [GitHub](https://github.com/Azure/azure-kusto-node/tree/master/azure-kusto-ingest)        |    [Npm](https://www.npmjs.com/package/azure-arm-kusto/v/2.0.0)        |
|    Python        |    [Pypi](https://pypi.org/project/azure-kusto-ingest/)    [GitHub](https://github.com/Azure/azure-kusto-python/tree/master/azure-kusto-data)        |    [Pypi](https://pypi.org/project/azure-kusto-data/)      [GitHub](https://github.com/Azure/azure-kusto-python/tree/master/azure-kusto-ingest)        |    [Pypi](https://pypi.org/project/azure-mgmt-kusto/0.3.0/)        |
|    R        |    [CRAN](https://cran.r-project.org/web/packages/AzureKusto/index.html)               |             |            |
|    Go        |    [GitHub](https://github.com/Azure/azure-kusto-go)        |    [GitHub](https://github.com/Azure/azure-kusto-go/tree/master/kusto/ingest)        |        [GitHub](https://github.com/Azure/azure-sdk-for-go/tree/master/services/kusto/mgmt/2019-01-21/kusto)        |
|    Ruby        |             |             |    [Gemas](https://rubygems.org/gems/azure_mgmt_kusto/versions/0.17.1)         |
|    PowerShell        |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/)        |    [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/)        |    [Paquete](https://www.powershellgallery.com/packages/Az.Kusto/)         |
|    Azure CLI        |             |             |    [Azure Cli](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest)         |
|    REST API        |    [REST](rest/index.md)        |    [REST](rest/index.md)        |     [GitHub](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/azure-kusto/resource-manager/Microsoft.Kusto)         |
|    TypeScript        |             |             |        [Npm](https://www.npmjs.com/package/@azure/arm-kusto/v/2.0.0)        |
|      |      |      |      |


## <a name="tools-and-integrations"></a>Herramientas e integraciones

* LightIngest: [Nuget](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/) 
* Kit de ingesta con un solo clic: [GitHub](https://github.com/Azure/azure-kusto-ingestion-tools) 
* Kafka: [GitHub](https://github.com/Azure/kafka-sink-azure-kustoLogstash)
* Logstash: [GitHub](https://github.com/Azure/logstash-output-kusto) 
* Chispa: [Maven](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/spark-kusto-connector)
---
title: 'Herramientas de Azure Data Explorer: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describen las herramientas de Azure Data Explorer en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: e9b274ae129f7cd15ba30edb24f8432c738f55ab
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490657"
---
# <a name="azure-data-explorer-tools"></a>Herramientas de Azure Data Explorer

## <a name="ad-hoc-query-tools"></a>Herramientas de consulta ad hoc


* [Kusto.Explorer](./kusto-explorer.md): la herramienta de escritorio principal para consultar y controlar Kusto
* [Interfaz de usuario web](https://docs.microsoft.com/azure/data-explorer/web-query-data): interfaz de usuario web para consultar Kusto

## <a name="visualizations-dashboards-and-reporting-tools"></a>Visualizaciones, paneles y herramientas de informes


* [Azure Notebooks](https://docs.microsoft.com/azure/data-explorer/azure-notebooks): Azure Notebooks se usa para analizar datos en Azure Data Explorer.
* Excel
    * [Consulta en blanco de Excel](https://docs.microsoft.com/azure/data-explorer/excel-blank-query): agregue una consulta de Kusto como origen de datos de Excel
    * [Conector de Excel](https://docs.microsoft.com/azure/data-explorer/excel-connector): un conector de Excel para Azure Data Explorer 

* PowerBI

   * [Procedimientos recomendados de PowerBI](https://docs.microsoft.com/azure/data-explorer/power-bi-best-practices)
   * [Conector de PowerBI](https://docs.microsoft.com/azure/data-explorer/power-bi-connector)
   * [Consulta importada de PowerBI](https://docs.microsoft.com/azure/data-explorer/power-bi-imported-query) 
   * [Consulta SQL de PowerBI](https://docs.microsoft.com/azure/data-explorer/power-bi-sql-query)

* [Grafana](https://docs.microsoft.com/azure/data-explorer/grafana)

## <a name="orchestration-tools"></a>Herramientas de orquestación


* Microsoft Flow
    * [Conector de Microsoft Flow](https://docs.microsoft.com/azure/data-explorer/flow)
    * [Ejemplos de uso del conector de Microsoft Flow](https://docs.microsoft.com/azure/data-explorer/flow-usage)
* [Microsoft Logic App](./logicapps.md): ejecute consultas de Kusto automáticamente como parte de [Microsoft Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps)



## <a name="data-ingestion-tools"></a>Herramientas de ingesta de datos


* [LightIngest](https://docs.microsoft.com/azure/data-explorer/lightingest): utilidad de ayuda para la ingesta de datos ad-hoc en Azure Data Explorer
 



## <a name="source-control-integration-tools"></a>Herramientas de integración de control de código fuente

* [Azure Pipelines](https://docs.microsoft.com/azure/data-explorer/devops): invoque comandos de control como parte de su canalización
* [Sincronizar Kusto](./synckusto.md): sincronice las funciones almacenadas de Kusto con Git

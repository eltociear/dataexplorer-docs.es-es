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
ms.openlocfilehash: ce7751d7ac60d23f9ffa0fc84992050fe1036131
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83370484"
---
# <a name="azure-data-explorer-tools"></a>Herramientas de Azure Data Explorer

## <a name="ad-hoc-query-tools"></a>Herramientas de consulta ad hoc


* [Kusto.Explorer](./kusto-explorer.md): la herramienta de escritorio principal para consultar y controlar Kusto
* [Interfaz de usuario web](../../web-query-data.md): interfaz de usuario web para consultar Kusto

## <a name="visualizations-dashboards-and-reporting-tools"></a>Visualizaciones, paneles y herramientas de informes


* [Azure Notebooks](../../azure-notebooks.md): Azure Notebooks se usa para analizar datos en Azure Data Explorer.
* Excel
    * [Consulta en blanco de Excel](../../excel-blank-query.md): agregue una consulta de Kusto como origen de datos de Excel
    * [Conector de Excel](../../excel-connector.md): un conector de Excel para Azure Data Explorer 

* PowerBI

   * [Procedimientos recomendados de PowerBI](../../power-bi-best-practices.md)
   * [Conector de PowerBI](../../power-bi-connector.md)
   * [Consulta importada de PowerBI](../../power-bi-imported-query.md) 
   * [Consulta SQL de PowerBI](../../power-bi-sql-query.md)

* [Grafana](../../grafana.md)

## <a name="orchestration-tools"></a>Herramientas de orquestación


* Microsoft Flow
    * [Conector de Microsoft Flow](../../flow.md)
    * [Ejemplos de uso del conector de Microsoft Flow](../../flow-usage.md)
* [Microsoft Logic App](./logicapps.md): ejecute consultas de Kusto automáticamente como parte de [Microsoft Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps)



## <a name="data-ingestion-tools"></a>Herramientas de ingesta de datos


* [LightIngest](../../lightingest.md): utilidad de ayuda para la ingesta de datos ad-hoc en Azure Data Explorer
 



## <a name="source-control-integration-tools"></a>Herramientas de integración de control de código fuente

* [Azure Pipelines](../../devops.md): invoque comandos de control como parte de su canalización
* [Sincronizar Kusto](./synckusto.md): sincronice las funciones almacenadas de Kusto con Git

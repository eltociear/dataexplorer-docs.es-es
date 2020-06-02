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
ms.openlocfilehash: b6bc95158c1dd161a17572342c6a99bdf9d37235
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84257916"
---
# <a name="azure-data-explorer-tools"></a>Herramientas de Azure Data Explorer

## <a name="ad-hoc-query-tools"></a>Herramientas de consulta ad hoc

* Kusto.Explorer
   * [Instalación e interfaz de usuario de Kusto.Explorer](./kusto-explorer.md): la herramienta de escritorio principal para consultar y controlar Kusto
   * [Uso de Kusto.Explorer](./kusto-explorer-using.md)
   * [Solución de problemas de Kusto.Explorer](kusto-explorer-troubleshooting.md)
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
* [Conector de código abierto K2Bridge](../../k2bridge.md): visualización de datos de Azure Data Explorer en Kibana

## <a name="orchestration-tools"></a>Herramientas de orquestación


* Microsoft Flow
    * [Conector de Microsoft Flow](../../flow.md)
    * [Ejemplos de uso del conector de Microsoft Flow](../../flow-usage.md)
* [Microsoft Logic App](./logicapps.md): ejecute consultas de Kusto automáticamente como parte de [Microsoft Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps)



## <a name="data-ingestion-tools"></a>Herramientas de ingesta de datos


* [LightIngest](../../lightingest.md): utilidad de ayuda para la ingesta de datos ad-hoc en Azure Data Explorer
* [Ingesta de un clic](../../ingest-data-one-click.md): herramienta para ingerir datos rápidamente y sugerir automáticamente tablas y estructuras de asignación
* [Azure Data Factory](azure-data-factory.md)


## <a name="source-control-integration-tools"></a>Herramientas de integración de control de código fuente

* [Azure Pipelines](../../devops.md): invoque comandos de control como parte de su canalización
* [Sincronizar Kusto](./synckusto.md): sincronice las funciones almacenadas de Kusto con Git

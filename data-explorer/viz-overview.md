---
title: Visualización de datos de Azure Data Explorer
description: Obtenga información sobre las distintas formas en que puede visualizar los datos de Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/02/2020
ms.openlocfilehash: b1351ceb9fe4b81a818ca41728a588dddfb4c5a2
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294685"
---
# <a name="data-visualization-with-azure-data-explorer"></a>Visualización de datos con Azure Data Explorer 

Azure Data Explorer es un servicio de exploración de datos altamente escalable para los datos de registro y telemetría que se usa para compilar soluciones complejas de análisis para grandes cantidades de datos. Azure Data Explorer se integra con varias herramientas de visualización, por lo que puede visualizar los datos y compartir los resultados en toda la organización. Estos datos se pueden convertir en información práctica que tendrá impacto en su empresa.

La generación de informes y visualización de los datos es un paso crítico del proceso de análisis de datos. Azure Data Explorer admite muchos servicios de inteligencia empresarial, por lo que puede usar el servicio que mejor se adapte a su escenario y presupuesto.

## <a name="azure-data-explorer-dashboards"></a>Paneles de Azure Data Explorer

Los paneles de Azure Data Explorer son una aplicación web que permite ejecutar consultas y compilar paneles en la aplicación web independiente, la [interfaz de usuario web](web-query-data.md). Los paneles de Azure Data Explorer ofrecen tres ventajas principales:

* Exportan de forma nativa las consultas de la interfaz de usuario Web a los paneles de Azure Data Explorer. 
* Exploran los datos en la interfaz de usuario web.
* Mejor rendimiento de la representación del panel.

Para más información, consulte [Visualización de datos con los paneles de Azure Data Explorer](azure-data-explorer-dashboards.md).

## <a name="kusto-query-language-visualizations"></a>Visualizaciones del lenguaje de consulta Kusto

El [`render operator`](kusto/query/renderoperator.md) del lenguaje de consulta Kusto ofrece varios tipos de visualizaciones, como tablas, gráficos circulares y gráficos de barras para mostrar los resultados de la consulta. Las visualizaciones de las consultas resultan útiles para la previsión y la detección de anomalías, el aprendizaje automático, etc.

## <a name="power-bi"></a>Power BI

Azure Data Explorer brinda la funcionalidad para conectarse a [Power BI](https://powerbi.microsoft.com) con varios métodos: 

  * [Built-in native Power BI connector](power-bi-connector.md) (Conector nativo de Power BI integrado)

  * [Query import from Azure Data Explorer into Power BI](power-bi-imported-query.md) (Importación de consultas desde Azure Data Explorer a Power BI)
 
  * [SQL query](power-bi-sql-query.md)

## <a name="microsoft-excel"></a>Microsoft Excel

Azure Data Explorer brinda la funcionalidad para conectarse a [Microsoft Excel](https://products.office.com/excel) con el [conector nativo de Excel integrado](excel-connector.md) o de [importar una consulta](excel-blank-query.md) desde Azure Data Explorer a Excel.

## <a name="grafana"></a>Grafana

[Grafana](https://grafana.com) proporciona un complemento para Azure Data Explorer que le permite visualizar los datos desde Azure Data Explorer. Puede [configurar Azure Data Explorer como un origen de datos para Grafana y, luego, visualizar los datos](grafana.md). 

## <a name="kibana"></a>Kibana

Azure Data Explorer permite conectarse a [Kibana](https://www.elastic.co/guide/en/kibana/6.8/discover.html) con K2Bridge, un conector de código abierto. Debe [configurar Azure Data Explorer como un origen de datos para Kibana y, luego, visualizar los datos](k2bridge.md).

## <a name="odbc-connector"></a>Conector ODBC

Azure Data Explorer proporciona un [conector para la conectividad abierta de bases de datos (ODBC)](connect-odbc.md), por lo que cualquier aplicación compatible con ODBC se puede conectar a Azure Data Explorer.

## <a name="tableau"></a>Tableau

Azure Data Explorer brinda la funcionalidad para conectarse a [Tableau](https://www.tableau.com) mediante el [conector ODBC](connect-odbc.md) y, luego, [visualizar los datos en Tableau](tableau.md).

## <a name="qlik"></a>Qlik

Azure Data Explorer brinda la funcionalidad para conectarse a [Qlik](https://www.qlik.com) mediante el [conector ODBC](connect-odbc.md) y, luego, crear paneles de Qlik Sense y visualizar los datos. En el siguiente vídeo, descubrirá cómo visualizar los datos de Azure Data Explorer con Qlik. 

> [!VIDEO https://www.youtube.com/embed/nhWIiBwxjjU]  

## <a name="sisense"></a>Sisense

Azure Data Explorer brinda la funcionalidad para conectarse a [Sisense](https://www.sisense.com) mediante el conector JDBC. Puede [configurar Azure Data Explorer como un origen de datos para Sisense y, luego, visualizar los datos](sisense.md).

## <a name="redash"></a>Redash

Puede usar [Redash](https://redash.io/) para compilar paneles y visualizar los datos. [Configure Azure Data Explorer como un origen de datos para Redash y, luego, visualice los datos](redash.md).

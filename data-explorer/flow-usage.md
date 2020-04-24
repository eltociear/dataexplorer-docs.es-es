---
title: Ejemplos de uso del conector de flujos de Microsoft Azure Data Explorer (versión preliminar)
description: Obtenga información sobre algunos ejemplos de uso habituales del conector de Microsoft Flow.
author: orspod
ms.author: orspodek
ms.reviewer: dorcohen
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/15/2020
ms.openlocfilehash: 05ff0e41347d56cc1e204defa8d5f250a881c84f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493448"
---
# <a name="microsoft-flow-connector-preview-usage-examples"></a>Ejemplos de uso del conector de Microsoft Flow (versión preliminar)

El conector de flujos de Azure Data Explorer permite a Azure Data Explorer usar las [funciones de flujo de Microsoft Power Automate](https://flow.microsoft.com/) para ejecutar consultas y comandos Kusto automáticamente como parte de una tarea programada o desencadenada. En este documento hay varios ejemplos de uso habituales del conector de Microsoft Flow.

Para obtener más información, consulte [Conector de Microsoft Flow (versión preliminar)](flow.md).

* [Conector de Microsoft Flow y SQL](#microsoft-flow-connector-and-sql)
* [Insertar datos en un conjunto de datos de Power BI](#push-data-to-power-bi-dataset)
* [Consultas condicionales](#conditional-queries)
* [Enviar por correo electrónico varios gráficos de flujos de Azure Data Explorer](#email-multiple-azure-data-explorer-flow-charts)

## <a name="microsoft-flow-connector-and-sql"></a>Conector de Microsoft Flow y SQL

Use el conector de Microsoft Flow para consultar los datos y agregarlos a una base de datos SQL.

> [!Note]
> La inserción SQL se efectúa por separado para cada fila. Use el conector de Microsoft Flow solo para pequeñas cantidades de datos de salida. 

![](./media/flow-usage/flow-sqlexample.png)

> [!IMPORTANT]
> En el campo *Nombre del clúster*, escriba la dirección URL del clúster.

## <a name="push-data-to-power-bi-dataset"></a>Insertar datos en un conjunto de datos de Power BI

El conector de Microsoft Flow se puede usar con el conector de Power BI para insertar datos de consultas Kusto en conjuntos de datos de streaming de Power BI.

1. Cree una acción Ejecutar la consulta y mostrar los resultados.
1. Seleccione **Nuevo paso**.
1. Seleccione **Agregar una acción** y busque "Power BI".
1. Seleccione **Power BI**.
1. Seleccione **Agregar filas a un conjunto de datos**. 

    ![Conector de flujos de Power BI](./media/flow-usage/flow-powerbiconnector.png)
1. Introduzca el **área de trabajo**, el **conjunto de datos** y la **tabla** en la que se van a insertar los datos.
1. En el cuadro de diálogo del contenido dinámico, agregue una carga útil que contenga el esquema del conjunto de datos y los resultados de la consulta Kusto pertinentes.

    ![Campos de flujos Power BI](./media/flow-usage/flow-powerbifields.png)

El flujo aplica automáticamente la acción de Power BI para cada fila de la tabla de resultados de la consulta Kusto. 

![Acción de flujo de Power BI para cada fila](./media/flow-usage/flow-powerbiforeach.png)

## <a name="conditional-queries"></a>Consultas condicionales

Los resultados de las consultas Kusto se pueden usar como entrada o como condiciones para las siguientes acciones de flujo.

En el siguiente ejemplo se hace una consulta a Kusto de los incidentes que se han producido el último día. Para cada incidente resuelto se publica un mensaje de slack y se crea una notificación de inserción.
En cuanto a los incidentes que siguen activos, se pide más información a Kusto sobre incidentes similares. Envía esa información por correo electrónico y abre una tarea de TFS relacionada.

Siga estas instrucciones para crear un flujo similar:

1. Cree una acción Ejecutar la consulta y mostrar los resultados.
1. Seleccione **Nuevo paso**.
1. Seleccione **Control de condición**.
1. En la ventana de contenido dinámico, seleccione el parámetro que quiere usar como condición para las siguientes acciones.
1. Seleccione el tipo de *relación* y *valor* para establecer una condición específica en el parámetro especificado.

    [![](./media/flow-usage/flow-condition.png "Flow conditions")](./media/flow-usage/flow-condition.png#lightbox)

    El flujo aplica esta condición en cada fila de la tabla de resultados de la consulta.
1. Agregue acciones para cuando la condición sea true y false.

    [![](./media/flow-usage/flow-conditionactions.png "Flow condition actions")](./media/flow-usage/flow-conditionactions.png#lightbox)

Puede usar los valores de los resultados de la consulta Kusto como entrada para las siguientes acciones. Seleccione los valores de los resultados de la ventana de contenido dinámico.
En el siguiente ejemplo se ha agregado una acción Slack - Publicar mensaje y una acción Visual Studio - Crear un elemento de trabajo que contienen datos de la consulta Kusto.

![Acción Slack - Publicar mensaje](./media/flow-usage/flow-slack.png)

![Acción de Visual Studio](./media/flow-usage/flow-visualstudio.png)

En este ejemplo, si un incidente sigue activo, consulte de nuevo a Kusto para obtener información sobre cómo se han resuelto en el pasado los incidentes del mismo origen.

![Consulta de condición de flujo](./media/flow-usage/flow-conditionquery.png)

> [!IMPORTANT]
> En el campo *Nombre del clúster*, escriba la dirección URL del clúster.

Visualice esta información en forma de gráfico circular y envíelo al equipo por correo electrónico.

![Correo electrónico de condición de flujo](./media/flow-usage/flow-conditionemail.png)

## <a name="email-multiple-azure-data-explorer-flow-charts"></a>Enviar por correo electrónico varios gráficos de flujos de Azure Data Explorer

1. Cree un flujo con el desencadenador de periodicidad y defina el intervalo del flujo y la frecuencia. 
1. Agregue un paso nuevo, con una o varias acciones Kusto - Ejecutar la consulta y visualizar los resultados. 

    ![Ejecutar varias consultas en un flujo](./media/flow-usage/flow-severalqueries.png)
1. Defina los siguientes campos para cada acción Kusto - Ejecutar la consulta y visualizar los resultados:
    * Dirección URL del clúster
    * Nombre de la base de datos
    * Tipo de consulta y gráfico (tabla HTML, gráfico circular, gráfico de tiempo, gráfico de barras o introduzca un valor personalizado).

    ![Visualizar los resultados con varios datos adjuntos](./media/flow-usage/flow-visualizeresultsmultipleattachments.png)

1. Agregue una acción "Enviar un correo electrónico" (v2): 
    1. En la sección del cuerpo, seleccione el icono de la vista Código.
    1. En el campo **Cuerpo**, inserte el elemento BodyHtml necesario para que el resultado visualizado de la consulta se incluya en el cuerpo del correo electrónico.
    1. Para agregar datos adjuntos al correo electrónico, agregue el nombre y el contenido de los datos adjuntos.
    
    ![Enviar por correo electrónico varios datos adjuntos](./media/flow-usage/flow-email-multiple-attachments.png)

    Para obtener instrucciones completas sobre cómo crear una acción de correo electrónico, consulte [Email Kusto query results](flow.md#email-kusto-query-results) (Enviar los resultados de la consulta Kusto por correo electrónico). 

Resultados:

[![](./media/flow-usage/flow-resultsmultipleattachments.png "Results of multiple attachments")](./media/flow-usage/flow-resultsmultipleattachments.png#lightbox)

[![](./media/flow-usage/flow-resultsmultipleattachments2.png "Results of multiple attachments")](./media/flow-usage/flow-resultsmultipleattachments2.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [conector de aplicaciones lógicas de Microsoft Azure Explorer](kusto/tools/logicapps.md), que es otra forma de ejecutar consultas y comandos Kusto automáticamente como parte de una tarea programada o desencadenada.

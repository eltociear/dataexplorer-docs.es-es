---
title: Ejemplos de uso del conector de Azure Data Explorer para Power Automate (versión preliminar)
description: Conozca algunos ejemplos de uso del conector de Azure Data Explorer para Power Automate
author: orspod
ms.author: orspodek
ms.reviewer: dorcohen
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/15/2020
ms.openlocfilehash: 0ecf0124051b6c003e056263afb6a3c5aa9ddb81
ms.sourcegitcommit: 98eabf249b3f2cc7423dade0f386417fb8e36ce7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82868712"
---
# <a name="usage-examples-for-azure-data-explorer-connector-to-power-automate-preview"></a>Ejemplos de uso del conector de Azure Data Explorer para Power Automate (versión preliminar)

El conector de flujo de Azure Data Explorer permite a Azure Data Explorer usar las funcionalidades de flujo de [Microsoft Power Automate](https://flow.microsoft.com/). Puede ejecutar consultas y comandos de Kusto automáticamente como parte de una tarea programada o desencadenada. En este artículo se incluyen varios ejemplos de uso comunes del conector de flujo.

Para más información, consulte [Conector de flujo de Azure Data Explorer (versión preliminar)](flow.md).

## <a name="flow-connector-and-your-sql-database"></a>Conector de flujo y base de datos SQL

Utilice el conector de flujo para consultar los datos y agregarlos a una base de datos SQL.

> [!Note]
> Utilice el conector de flujo solo para pequeñas cantidades de datos de salida. La operación de inserción SQL se efectúa por separado para cada fila. 

![Captura de pantalla de la consulta de datos mediante el conector de flujo](./media/flow-usage/flow-sqlexample.png)

> [!IMPORTANT]
> En el campo **Nombre del clúster**, escriba la dirección URL del clúster.

## <a name="push-data-to-a-microsoft-power-bi-dataset"></a>Inserción de datos en un conjunto de datos de Microsoft Power BI

El conector de flujo se puede usar con el conector de Power BI para insertar datos de consultas Kusto en conjuntos de datos de streaming de Power BI.

1. Cree una acción **Ejecutar la consulta y mostrar los resultados**.
1. Seleccione **Nuevo paso**.
1. Seleccione **Agregar una acción** y busque "Power BI".
1. Seleccione **Power BI** > **Agregar filas a un conjunto de datos**. 

    ![Captura de pantalla del conector de Power BI](./media/flow-usage/flow-powerbiconnector.png)

1. Introduzca el **área de trabajo**, el **conjunto de datos** y la **tabla** en la que se van a insertar los datos.
1. En el cuadro de diálogo del contenido dinámico, agregue una **carga útil** que contenga el esquema del conjunto de datos y los resultados de la consulta Kusto pertinentes.

    ![Captura de pantalla de los campos de Power BI](./media/flow-usage/flow-powerbifields.png)

El flujo aplica automáticamente la acción de Power BI a cada fila de la tabla de resultados de la consulta de Kusto. 

![Captura de pantalla de la acción de Power BI para cada fila](./media/flow-usage/flow-powerbiforeach.png)

## <a name="conditional-queries"></a>Consultas condicionales

Puede usar los resultados de las consultas de Kusto como entrada o como condiciones para las siguientes acciones de flujo.

En el siguiente ejemplo se hace una consulta a Kusto de los incidentes que se han producido el último día. Para cada incidente resuelto, se publica un mensaje de Slack y se crea una notificación de inserción.
En cuanto a los incidentes que siguen activos, se pide a Kusto más información sobre incidentes similares. Envía esa información por correo electrónico y abre una tarea relacionada en Azure DevOps Server.

Siga estas instrucciones para crear un flujo similar:

1. Cree una acción **Ejecutar la consulta y mostrar los resultados**.
1. Seleccione **Nuevo paso** > **Control de condición** .
1. En la ventana de contenido dinámico, seleccione el parámetro que quiere usar como condición para las siguientes acciones.
1. Seleccione el tipo de *relación* y el *valor* para establecer una condición específica en el parámetro concreto.

    [![](./media/flow-usage/flow-condition.png "Screenshot of flow conditions")](./media/flow-usage/flow-condition.png#lightbox)

    El flujo aplica esta condición en cada fila de la tabla de resultados de la consulta.
1. Agregue acciones para cuando la condición sea true y false.

    [![](./media/flow-usage/flow-conditionactions.png "Screenshot of flow condition actions")](./media/flow-usage/flow-conditionactions.png#lightbox)

Puede usar los valores de los resultados de la consulta Kusto como entrada para las siguientes acciones. Seleccione los valores de los resultados de la ventana de contenido dinámico.
En el siguiente ejemplo se ha agregado una acción **Slack - Publicar mensaje** y una acción **Visual Studio - Crear un elemento de trabajo** que contienen datos de la consulta de Kusto.

![Pantalla de la acción Slack - Publicar mensaje](./media/flow-usage/flow-slack.png)

![Captura de pantalla de la acción de Visual Studio](./media/flow-usage/flow-visualstudio.png)

En este ejemplo, si un incidente sigue activo, consulte de nuevo a Kusto para obtener información sobre cómo se han resuelto en el pasado los incidentes del mismo origen.

![Captura de pantalla de la consulta de condición del flujo](./media/flow-usage/flow-conditionquery.png)

> [!IMPORTANT]
> En el campo **Nombre del clúster**, escriba la dirección URL del clúster.

Visualice esta información en forma de gráfico circular y envíelo al equipo por correo electrónico.

![Captura de pantalla del correo electrónico de condición de flujo](./media/flow-usage/flow-conditionemail.png)

## <a name="email-multiple-azure-data-explorer-flow-charts"></a>Envío por correo electrónico de varios gráficos de flujo de Azure Data Explorer

1. Cree un flujo con el desencadenador de periodicidad y defina el intervalo y la frecuencia del flujo. 
1. Agregue un paso nuevo, con una o varias acciones **Kusto: Ejecutar la consulta y visualizar los resultados**. 

    ![Captura de pantalla de la ejecución de varias consultas en un flujo](./media/flow-usage/flow-severalqueries.png)

1. Para cada acción **Kusto - Ejecutar la consulta y visualizar los resultados**, define los campos siguientes:
    * Dirección URL del clúster
    * Nombre de base de datos
    * Tipo de consulta y gráfico (por ejemplo, tabla HTML, gráfico circular, gráfico de tiempo, gráfico de barras o un valor personalizado)

    ![Captura de pantalla de la visualización de resultados con varios datos adjuntos](./media/flow-usage/flow-visualizeresultsmultipleattachments.png)

1. Agregue una acción **Enviar un correo electrónico (v2)** : 
    1. En la sección del cuerpo, seleccione el icono de la vista Código.
    1. En el campo **Cuerpo**, inserte el elemento **BodyHtml** necesario para que el resultado visualizado de la consulta se incluya en el cuerpo del correo electrónico.
    1. Para agregar datos adjuntos al correo electrónico, agregue el **nombre** y el **contenido** de los datos adjuntos.
    
    ![Captura de pantalla del envío por correo electrónico de varios datos adjuntos](./media/flow-usage/flow-email-multiple-attachments.png)

    Para más información sobre cómo crear una acción de correo electrónico, consulte [Resultados de la consulta Kusto por correo electrónico](flow.md#email-kusto-query-results). 

Resultados:

[![](./media/flow-usage/flow-resultsmultipleattachments.png "Screenshot of results of multiple attachments, visualized as a pie chart and bar chart")](./media/flow-usage/flow-resultsmultipleattachments.png#lightbox)

[![](./media/flow-usage/flow-resultsmultipleattachments2.png "Screenshot of results of multiple attachments, visualized as a time chart")](./media/flow-usage/flow-resultsmultipleattachments2.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el [conector de aplicaciones lógicas de Microsoft Kusto](kusto/tools/logicapps.md), que es otra forma de ejecutar consultas y comandos Kusto automáticamente como parte de una tarea programada o desencadenada.


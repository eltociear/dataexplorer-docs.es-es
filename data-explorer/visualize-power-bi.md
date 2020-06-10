---
title: 'Tutorial: Visualización de datos desde Azure Data Explorer en Power BI'
description: En este tutorial, aprenderá a conectarse al Explorador de datos de Azure con Power BI y visualizar sus datos.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: tutorial
ms.date: 06/02/2020
ms.openlocfilehash: bde567a3c41a226ca0a8304d7ab4ed42f3fa65bf
ms.sourcegitcommit: 8953d09101f4358355df60ab09e55e71bc255ead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2020
ms.locfileid: "84420924"
---
# <a name="tutorial-visualize-data-from-azure-data-explorer-in-power-bi"></a>Tutorial: Visualización de datos desde Azure Data Explorer en Power BI

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Power BI es una solución de análisis de negocios que le permite visualizar sus datos y compartir los resultados en su organización. En este tutorial, aprenderá primero a representar objetos visuales en el Explorador de datos de Azure. A continuación, se conectará al Explorador de datos de Azure con Power BI, creará un informe basado en datos de ejemplo y publicará el informe en el servicio Power BI.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar. Si no está registrado para Power BI Pro, [regístrese para una evaluación gratuita](https://app.powerbi.com/signupredirect?pbi_source=web) antes de comenzar.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Representar objetos visuales en el Explorador de datos de Azure
> * Conectarse el Explorador de datos de Azure en Power BI Desktop
> * Trabajar con datos en Power BI Desktop
> * Crear un informe con objetos visuales
> * Publicar y compartir el informe

## <a name="prerequisites"></a>Requisitos previos

Además de las suscripciones de Azure y Power BI, necesita lo siguiente para completar este tutorial:

* [Base de datos y clúster de prueba](create-cluster-database-portal.md)

* [Los datos de ejemplo de StormEvents](ingest-sample-data.md). [!INCLUDE [data-explorer-storm-events](includes/data-explorer-storm-events.md)]

* [Power BI Desktop](https://powerbi.microsoft.com/get-started/) (seleccione **DESCARGAR GRATIS**)

## <a name="render-visuals-in-azure-data-explorer"></a>Representar objetos visuales en el Explorador de datos de Azure

Antes de pasar a Power BI, echemos un vistazo a cómo representar objetos visuales en el Explorador de datos de Azure. Esto es ideal para algunos análisis rápidos.

1. Inicie sesión en [https://dataexplorer.azure.com](https://dataexplorer.azure.com).

1. En el panel izquierdo, seleccione la base de datos de prueba que contiene los datos de ejemplo de StormEvents.

1. Pegue la siguiente consulta en la ventana derecha y seleccione **Ejecutar**.

    ```Kusto
    StormEvents
    | summarize event_count=count() by State
    | where event_count > 1800
    | project State, event_count
    | sort by event_count
    | render columnchart
    ```

    Esta consulta cuenta eventos de tiempo por estado. A continuación, representa un gráfico de columnas para todos los estados que tienen más de 1800 eventos de tiempo.

    ![Gráfico de columnas de eventos](media/visualize-power-bi/events-column-chart.png)

1. Pegue la siguiente consulta en la ventana derecha y seleccione **Ejecutar**.

    ```Kusto
    StormEvents
    | where State == "WASHINGTON" and StartTime >= datetime(2007-07-01) and StartTime <= datetime(2007-07-31)
    | summarize StormCount = count() by EventType
    | render piechart
    ```

    Esta consulta cuenta eventos de tiempo por tipo durante el mes de julio en el estado de Washington. A continuación, representa un gráfico circular que muestra el porcentaje de cada tipo de evento.

    ![Gráfico circular de eventos](media/visualize-power-bi/events-pie-chart.png)

Ahora es el momento de echar un vistazo a Power BI, pero puede hacer mucho más con los objetos visuales en el Explorador de datos de Azure.

## <a name="connect-to-azure-data-explorer"></a>Conectarse al Explorador de datos de Azure

Conéctese ahora al Explorador de datos de Azure en Power BI Desktop.

1. En la pestaña **Inicio** de Power BI Desktop, seleccione **Obtener datos** y luego **Más**.

    ![Obtener datos](media/visualize-power-bi/get-data-more.png)

1. Busque *Azure Data Explorer*, seleccione **Azure Data Explorer** y, a continuación, **Conectar**.

    ![Buscar y obtener datos](media/visualize-power-bi/search-get-data.png)

1. En la pantalla **Conector de vista previa**, seleccione **Continuar**.

1. En la pantalla siguiente, escriba el nombre de su clúster y su base de datos de prueba. El clúster debe tener el formato `https://<ClusterName>.<Region>.kusto.windows.net`. Escriba *StormEvents* para el nombre de la tabla. Deje todas las demás opciones con valores predeterminados y seleccione **Aceptar**.

    ![Opciones de clúster, base de datos y tabla](media/visualize-power-bi/cluster-database-table.png)

1. En la pantalla Vista previa de los datos, seleccione **Editar**.

    La tabla se abre en el editor de Power Query, donde puede editar filas y columnas antes de importar los datos.

## <a name="work-with-data-in-power-bi-desktop"></a>Trabajar con datos en Power BI Desktop

Ahora que tiene una conexión al Explorador de datos de Azure, edita los datos en el editor de Power Query. Coloca filas con valores NULL en la columna **BeginLat** y coloca la columna JSON **StormSummary** por completo. Estas son operaciones sencillas, pero también puede realizar transformaciones complejas al importar los datos.

1. Seleccione la flecha de la columna **BeginLat**, desactive la casilla **NULL** y, a continuación, seleccione **Aceptar**.

    ![Filtrar columna](media/visualize-power-bi/filter-column.png)

1. Haga clic con el botón derecho en el encabezado de columna **StormSummary** y seleccione **Quitar**.

    ![Quitar columna](media/visualize-power-bi/remove-column.png)

1. En el panel **CONFIGURACIÓN DE LA CONSULTA**, cambie el nombre de *Consulta1* a *StormEvents*.

    ![Cambiar el nombre de la consulta](media/visualize-power-bi/query-name.png)

1. En la pestaña **Inicio** de la cinta, seleccione **Cerrar y aplicar**.

    ![Cerrar y aplicar](media/visualize-power-bi/close-apply.png)

    Power Query aplica sus cambios y, a continuación, importa los datos de ejemplo en un *modelo de datos*. En los siguientes pasos se le muestra cómo enriquecer ese modelo. Una vez más, este es solo un ejemplo para dar una idea de lo que es posible.

1. En el lado izquierdo de la ventana principal, seleccione la vista de datos.

    ![Vista de datos](media/visualize-power-bi/data-view.png)

1. En la pestaña **Modelado** de la cinta, seleccione **Nueva columna**.

    ![Nueva columna](media/visualize-power-bi/new-column.png)

1. Escriba la siguiente fórmula de expresiones de análisis de datos (DAX) en la barra de fórmulas y, a continuación, presione Entrar.

    ```DAX
    DurationHours = DATEDIFF(StormEvents[StartTime], StormEvents[EndTime], hour)
    ```

    ![Barra de fórmulas](media/visualize-power-bi/formula-bar.png)

    Esta fórmula crea la columna *DurationHours*, que calcula cuántas horas duró cada evento de tiempo. En la siguiente sección, usará esta columna en un objeto visual.

1. Desplácese al lado derecho de la tabla para ver la columna.

## <a name="create-a-report-with-visuals"></a>Crear un informe con objetos visuales

Ahora que los datos se importan y ha mejorado el modelo de datos, es el momento de crear un informe con objetos visuales. Agrega un gráfico de columnas basado en la duración del evento y un mapa que muestra daños derivados de recortes.

1. En el lado izquierdo de la ventana, seleccione la vista de datos.

    ![Vista de informe](media/visualize-power-bi/report-view.png)

1. En el panel **VISUALIZACIONES**, seleccione el gráfico de columnas agrupadas.

    ![Agregar gráfico de columnas](media/visualize-power-bi/add-column-chart.png)

    Un gráfico en blanco se agrega al lienzo.

    ![Gráfico en blanco](media/visualize-power-bi/blank-chart.png)

1. En la lista **CAMPOS**, seleccione **DurationHours** y **Estado**.

    ![Seleccionar campos](media/visualize-power-bi/select-fields.png)

    Ahora tiene un gráfico donde se muestran las horas totales de eventos de tiempo por estado en el transcurso de un año.

    ![Gráfico de columnas de duración](media/visualize-power-bi/duration-column-chart.png)

1. Haga clic en cualquier parte del lienzo fuera del gráfico de columnas.

1. En el panel **VISUALIZACIONES**, seleccione el mapa.

    ![Agregar mapa](media/visualize-power-bi/add-map.png)

1. En la lista **CAMPOS**, seleccione **CropDamage** y **Estado**. Cambie el tamaño del mapa para que pueda ver los estados de EE. UU. con claridad.

    ![Mapa de daños derivados de recortes](media/visualize-power-bi/crop-damage-map.png)

    El tamaño de las burbujas representa el valor en dólares de los daños derivados de recortes. Coloque el mouse sobre las burbujas para ver detalles.

1. Mueva y cambie el tamaño de los objetos visuales de modo que tenga un informe con un aspecto similar al de la siguiente imagen.

    ![Informe finalizado](media/visualize-power-bi/finished-report.png)

1. Guarde el informe con el nombre *storm-events.pbix*.

## <a name="publish-and-share-the-report"></a>Publicar y compartir el informe

Hasta este momento, todo el trabajo que ha realizado en Power BI ha sido local, mediante Power BI Desktop. Publique ahora el informe en el servicio Power BI, donde puede compartirlo con otros usuarios.

1. En Power BI Desktop, en la pestaña **Inicio** de la cinta, seleccione **Publicar**.

    ![Botón Publicar](media/visualize-power-bi/publish-button.png)

1. Si aún no ha iniciado sesión en Power BI, siga el proceso de inicio de sesión.

1. Seleccione **Mi área de trabajo** y, a continuación, **Seleccionar**.

    ![Selección del área de trabajo](media/visualize-power-bi/select-workspace.png)

1. Al finalizar la publicación, seleccione **Abrir storm-events.pbix en Power BI**.

    ![Publicación correcta](media/visualize-power-bi/publishing-succeeded.png)

    El informe se abre en el servicio, con los mismos objetos visuales y diseño que definió en Power BI Desktop.

1. En la esquina superior derecha del informe, seleccione **Compartir**.

    ![Botón Compartir](media/visualize-power-bi/share-button.png)

1. En la pantalla **Compartir informe**, agregue a un compañero de su organización, agregue una nota y, a continuación, seleccione **Compartir**.

    ![Compartir informe](media/visualize-power-bi/share-report.png)

    Si su compañero tiene los permisos adecuados, puede tener acceso al informe que compartió.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no desea guardar el informe que creó, solo tiene que eliminar el archivo *storm-events.pbix*. Si desea quitar el informe que publicó, siga estos pasos.

1. En **Mi área de trabajo**, desplácese hacia abajo hasta **INFORMES** y busque **storm-events**.

1. Seleccione los puntos suspensivos ( **. . .** ) situados junto a **storm-events** y, a continuación, seleccione **QUITAR**.

    ![Quitar informe](media/visualize-power-bi/remove-report.png)

1. Confirme la eliminación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Escritura de consultas](write-queries.md)

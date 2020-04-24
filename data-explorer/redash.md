---
title: Visualización de Azure Data Explorer con Redash
description: En este artículo, aprenderá a visualizar datos en Azure Data Explorer con el conector nativo de Redash.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 8237eb917f5123755898d281151ce34a08eb90db
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492372"
---
# <a name="visualize-data-from-azure-data-explorer-in-redash"></a>Visualización de datos desde Azure Data Explorer en Redash

[Redash](https://redash.io/) se conecta a los orígenes de datos y los consulta, y crea paneles para visualizar datos y compartirlos con los emparejamientos. En este artículo, aprenderá a configurar Azure Data Explorer como origen de datos de Redash y luego a visualizar datos.

## <a name="prerequisites"></a>Prerrequisitos

1. [Crear un clúster y una base de datos](create-cluster-database-portal.md).
1. Ingerir datos como se explica en [Ingesta de datos de ejemplo en Azure Data Explorer](ingest-sample-data.md). Para más opciones de ingesta, consulte la [información general sobre la ingesta](ingest-data-overview.md).

[!INCLUDE [data-explorer-configure-data-source](includes/data-explorer-configure-data-source.md)]

## <a name="create-azure-data-explorer-connector-in-redash"></a>Creación de un conector de Azure Data Explorer en Redash 

1. Inicie sesión en [Redash](https://www.redash.io/). Seleccione **Empecemos** para crear una cuenta.
1. En **Comencemos**, seleccione **Conectar un origen de datos**.

    ![Conexión de un origen de datos](media/redash/connect-data-source.png)

1. En la ventana **Crear un nuevo origen de datos** , seleccione **Azure Data Explorer (Kusto)**  y, luego, seleccione **Crear**. 

    ![Selección de un origen de datos de Azure Data Explorer](media/redash/select-adx-data-source.png)

1. En la ventana **Azure Data Explorer (Kusto)** , complete el siguiente formulario y seleccione **Crear**.

    ![Ventana de configuración Azure Data Explorer (Kusto)](media/redash/adx-settings-window.png)

1. En la ventana **Configuración**, seleccione **Guardar** y **Probar la conexión** para probar la conexión del origen de datos de **Azure Data Explorer (Kusto)** .

## <a name="create-queries-in-redash"></a>Creación de consultas en Redash

1. En la parte superior izquierda de Redash, seleccione **Crear** > **Consulta**. Haga clic en **Nueva consulta** y cambie el nombre de la consulta.

    ![Creación de una consulta](media/redash/create-query.png)

1. Escriba la consulta en el panel de edición superior y seleccione **Guardar** y **Ejecutar**. Seleccione **Publicar** para publicar la consulta para un uso futuro.

    ![Almacenamiento y ejecución de la consulta](media/redash/save-and-execute-query.png)

    En el panel izquierdo, puede ver el nombre de la conexión del origen de datos (**conector de github** en el flujo) en el menú desplegable y las tablas de la base de datos seleccionada. 

1. Vea los resultados de la consulta en el panel central inferior. Cree una visualización para ir con la consulta seleccionando el botón **Nueva visualización**.

    ![Nueva visualización](media/redash/new-visualization.png)

1. En la pantalla de visualización, seleccione una opción en **Tipo de visualización** y los campos pertinentes como **Columna X** y **Columna y**. **Guarde** la visualización.

    ![Configuración y almacenamiento de la visualización](media/redash/configure-visualization.png)

### <a name="create-a-query-using-a-parameter"></a>Creación de una consulta mediante un parámetro

1. Seleccione **Crear** > **Consulta** para crear una nueva consulta. Agregue un parámetro a ella mediante llaves {{}}. Seleccione **{{}}** para abrir la ventana **Agregar parámetro**. También puede seleccionar el *icono de configuración* para modificar los atributos de un parámetro existente y abrir la ventana **<nombre_del_parámetro>** . 

    ![Inserción del parámetro](media/redash/insert-parameter.png)

1. Asigne un nombre al parámetro. Seleccione **Tipo**: **Query Based Dropdown List** (Lista desplegable basada en consulta) del menú desplegable. Seleccione **Aceptar**.

    ![Lista desplegable basada en consultas](media/redash/query-based-dropdown-list.png)

    > [!NOTE]
    > La consulta utiliza varios valores, por lo que debe incluir la sintaxis `| where Type in ((split('{{Type}}', ',')))`. Para más información, vea el [operador in](kusto/query/inoperator.md). Esto da como resultado [varias opciones de parámetro de consulta en la aplicación de Redash](https://redash.io/help/user-guide/querying/query-parameters#Serialized-Multi-Select-Query-Parametersredash.io).

## <a name="create-a-dashboard-in-redash"></a>Creación de un panel en Redash

1. Para crear un panel, seleccione **Crear** > **Panel**. Como alternativa, seleccione el panel existente, **Paneles** > seleccione un panel de la lista.

    ![Crear panel](media/redash/create-dashboard.png)

1. En ventana **Nuevo panel**, asigne un nombre al panel y seleccione **Guardar**. En la ventana **<Nombre_del_panel>** , seleccione **Agregar widget** para crear un nuevo widget. 

1. En la ventana **Agregar widget**, seleccione el nombre de la consulta, **Choose Visualization** (Elegir visualización)
y **Parámetros**. Seleccione **Agregar al panel**.

   ![Elección de visualizaciones e incorporación al panel](media/redash/add-widget-window.png)

1. Seleccione **Edición finalizada** para completar la creación del panel.

1.  En el modo de edición del panel, seleccione **Use Dashboard Level Filters** (Usar filtros de nivel de panel) para usar el parámetro **Tipo** definido previamente.

    ![Finalización de la creación del panel](media/redash/complete-dashboard.png)

## <a name="next-steps"></a>Pasos siguientes

* [Write queries for Azure Data Explorer](write-queries.md) (Escritura de consultas del Explorador de datos de Azure)



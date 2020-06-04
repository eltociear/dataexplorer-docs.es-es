---
title: Visualización de datos con el panel de Azure Data Explorer
description: Aprenda a visualizar datos con el panel de Azure Data Explorer
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/26/2020
ms.openlocfilehash: 46e3821514ca5b06852f7e8428b12cf9f80e29a4
ms.sourcegitcommit: a562ce255ac706ca1ca77d272a97b5975235729d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83867799"
---
# <a name="visualize-data-with-azure-data-explorer-dashboards"></a>Visualización de datos con los paneles de Azure Data Explorer

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Azure Data Explorer proporciona una aplicación web que permite ejecutar consultas y crear paneles. Los paneles están disponibles en la aplicación web independiente, la [interfaz de usuario web](web-query-data.md). Azure Data Explorer también está integrado en otros servicios del panel, como [Power BI](power-bi-connector.md) y [Grafana](grafana.md).

Los paneles de Azure Data Explorer ofrecen tres ventajas principales:

* Exportan de forma nativa las consultas de la interfaz de usuario Web a los paneles de Azure Data Explorer. 
* Exploran los datos en la interfaz de usuario web.
* Mejor rendimiento de la representación del panel.

En la imagen siguiente se muestra un panel de Azure Data Explorer.

:::image type="content" source="media/adx-dashboards/dash.png" alt-text="Panel final":::

## <a name="create-a-dashboard"></a>Creación de un panel

1. En la barra de navegación, seleccione **Dashboards** (Paneles) y seleccione **New dashboard** (Nuevo panel).

    :::image type="content" source="media/adx-dashboards/new-dashboard.png" alt-text="New dashboard"::: (Nuevo panel)

1. Seleccione un nombre de panel y **Create** (Crear).

    :::image type="content" source="media/adx-dashboards/new-dashboard-popup.png" alt-text="Creación de un panel":::

## <a name="add-data-source"></a>Agregar origen de datos

Agregue los orígenes de datos necesarios para los paneles.

1. Seleccione el elemento de menú **Data sources** (Orígenes de datos) en la barra superior. Seleccione el botón **+ New data source** + Nuevo origen de datos en el panel **Data sources** (Orígenes de datos).

    :::image type="content" source="media/adx-dashboards/data-source.png" alt-text="Origen de datos":::

1. En el panel **Create new data source** (Crear nuevo origen de datos):
    1. Escriba el **identificador URI del clúster** o el nombre parcial que incluye la región y seleccione **Connect** (Conectar). 
    1. Seleccione la **Base de datos** de la lista desplegable.
    1. Utilice el valor predeterminado del campo **Data source name** (Nombre del origen de datos) o modifíquelo si fuera necesario. 
    1. Seleccione **Aplicar**.

    :::image type="content" source="media/adx-dashboards/data-source-pane.png" alt-text="Panel de orígenes de datos":::

## <a name="use-parameters"></a>Uso de parámetros

Los parámetros permiten usar los filtros de los paneles. Los parámetros mejoran considerablemente el rendimiento de la representación del panel y permiten usar valores de filtro lo antes posible en la consulta.

1. Seleccione **Parameters** (Parámetros) en la barra superior. Seleccione el botón **+ New parameter** (+ Parámetro nuevo) en el panel **Parameters** (Parámetros).

    :::image type="content" source="media/adx-dashboards/parameters.png" alt-text="Seleccionar nuevo parámetro":::

1. Escriba los valores de todos los campos obligatorios y seleccione **Done** (Listo).

    :::image type="content" source="media/adx-dashboards/parameter-pane.png" alt-text="Panel Parameters"::: (Parámetros)

|Campo  |Descripción |
|---------|---------|
|Escriba el **Parameter display name** (Nombre para mostrar del parámetro)    |   Nombre del parámetro que se muestra en el panel o en la tarjeta de edición.      |
|**Parameter type** (Tipo de parámetro)    |Uno de los siguientes:<ul><li>**Single selection** (Selección única): solo se puede seleccionar un valor en el filtro como entrada para el parámetro.</li><li>**Multiple selection** (Selección múltiple): se pueden seleccionar uno o varios valores en el filtro como entradas para el parámetro.</li><li>**Intervalo de tiempo**: permite crear parámetros adicionales para filtrar las consultas y los paneles en función del tiempo. De forma predeterminada, cada panel tiene un selector de intervalos de tiempo.</li></ul>    |
|**Nombre de la variable**     |   Nombre del parámetro que se va a utilizar en la consulta.      |
|**Tipo de datos**    |    Tipo de datos del valor del parámetro.     |
|**Pin as dashboard filter** (Anclar como filtro del panel)   |   Ancle el filtro basado en parámetros al panel o desánclelo del panel.       |
|**Origen**     |    Origen de los valores valor del parámetro: <ul><li>**Fixed values** (Valores fijos): valores del filtro estático especificados manualmente. </li><li>**Consultar** valores especificados de forma dinámica mediante una consulta de KQL.  </li></ul>    |
|**Add a "Select all" value** (Agregar un valor "Seleccionar todo")    |   Solo se aplica a los tipos de parámetro de selección única y selección múltiple. Se utiliza para recuperar los datos de todos los valores de los parámetros.      |

## <a name="add-query"></a>Adición de consulta

**Add Query** (Agregar consulta) utiliza fragmentos de código del lenguaje de consulta de Kusto para recuperar datos y representar objetos visuales. Cada consulta puede admitir un solo objeto visual.

1. Seleccione **Add Query** (Agregar consulta) en el lienzo del panel o en la barra de menús superior.

    :::image type="content" source="media/adx-dashboards/empty-dashboard-new-query.png" alt-text="Nueva consulta":::

1. En el panel **Query** (Consulta). 
    1. Seleccione el origen de datos en la lista desplegable.
    1. Escriba la consulta y seleccione **Run** (Ejecutar). 
    1. Seleccione **+ Add visual** (+ Agregar objeto visual).

    :::image type="content" source="media/adx-dashboards/initial-query.png" alt-text="Ejecutar consulta":::

1. En el panel **Visual formatting** (Formato visual), seleccione **Chart type** (Tipo de gráfico) para elegir el tipo de objeto visual. 
1. Asigne un nombre al objeto visual y seleccione **Apply changes** (Aplicar cambios) para anclar el objeto visual al panel.

    :::image type="content" source="media/adx-dashboards/add-visual.png" alt-text="Agregar un objeto visual a una consulta":::

1. Puede cambiar el tamaño del objeto visual y seleccionar **Save changes** (Guardar cambios) para guardar el panel.

    :::image type="content" source="media/adx-dashboards/save-dashboard.png" alt-text="Guardar el panel":::

## <a name="next-steps"></a>Pasos siguientes

* [Consulta de datos en Azure Data Explorer](web-query-data.md)
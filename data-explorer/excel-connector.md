---
title: Visualización de datos mediante el conector de Azure Data Explorer para Microsoft Excel
description: En este artículo, aprenderá a usar el conector de Excel para Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/30/2019
ms.openlocfilehash: 42f52581d8f2f80deb5d6250ed54ab64fc1ba4d3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493672"
---
# <a name="visualize-data-using-the-azure-data-explorer-connector-for-excel"></a>Visualización de datos mediante el conector de Azure Data Explorer para Excel

Azure Data Explorer proporciona dos opciones para conectarse a datos de Excel: usar el conector nativo o importar una consulta desde Azure Data Explorer. En este artículo se muestra cómo usar el conector nativo de Excel y cómo conectarse al clúster de Azure Data Explorer para obtener y visualizar los datos.

El conector nativo de Excel para Azure Data Explorer ofrece la posibilidad de exportar los resultados de consulta a Excel. También puede agregar una consulta de KQL como origen de datos de Excel para realizar visualizaciones o cálculos adicionales.

## <a name="define-kusto-query-as-an-excel-data-source-and-load-the-data-to-excel"></a>Definición de una consulta de Kusto como un origen de datos de Excel y carga de los datos en Excel

1. Abra **Microsoft Excel**.
1. En la pestaña **Datos**, seleccione **Obtener datos** > **Desde Azure** > **Desde Azure Data Explorer**.

    ![Obtención de datos de Azure Data Explorer](media/excel-connector/get-data-from-adx.png)

1. En la ventana **Azure Data Explorer (Kusto)** , complete los siguientes campos y seleccione **Aceptar**.

    ![Ventana de Azure Data Explorer (Kusto)](media/excel-connector/adx-connection-window.png)
    
    |Campo   |Descripción |
    |---------|---------|
    |**Clúster**   |   Nombre del clúster (obligatorio)      |    
    |**Base de datos**     |    Nombre de base de datos      |    
    |**Nombre de la tabla o consulta de Azure Data Explorer**    |     Nombre de la tabla o consulta de Azure Data Explorer.    | 
    
    **Opciones avanzadas:**

     |Campo   |Descripción |
    |---------|---------|
    |**Limitar el número de registro de resultados de consulta**     |     Limita el número de registros cargados en Excel.  |    
    |**Limitar el tamaño de los datos del resultado de una consulta en bytes**    |    Limita el tamaño de los datos.      |   
    |**Deshabilitar el truncamiento del conjunto de resultados**    |         |      
    |**Instrucciones SET adicionales (separadas por punto y coma)**    |    Agrega instrucciones `set` para aplicar al origen de datos.     |   

1.  En el panel **Navegador**, vaya a la tabla correcta. En el panel de vista previa de la tabla, seleccione **Transformar datos** para realizar cambios en los datos, o seleccione **Cargar** para cargarlos en Excel.

![Ventana de vista previa de la tabla](media/excel-connector/navigate-table-preview-window.png)

   > [!TIP]
   > Si ya se han especificado las opciones **Base de datos** o **Nombre de la tabla o consulta de Azure Data Explorer**, el panel correcto de vista previa de la tabla se abre automáticamente. 

## <a name="analyze-and-visualize-data-in-excel"></a>Análisis y visualización de los datos en Excel

Cuando los datos se carguen en Excel y estén disponibles en su hoja, puede analizarlos, resumirlos y visualizarlos mediante la creación de relaciones y objetos visuales. 

1.  En la pestaña **Diseño de tabla**, seleccione **Resumir con tabla dinámica**. En la ventana **Crear tabla dinámica**, seleccione la tabla pertinente y haga clic en **Aceptar**.

    ![Creación de tabla dinámica](media/excel-connector/create-pivot-table.png)

1. En el panel **Campos de tabla dinámica**, seleccione las columnas de tabla pertinentes para crear tablas de resumen. En el ejemplo siguiente, se seleccionan **EventId** y **State**.
    
    ![Selección de campos de tabla dinámica](media/excel-connector/pivot-table-pick-fields.png)

1. En la pestaña **Análisis de tabla dinámica**, seleccione **Gráfico dinámico** para crear objetos visuales basados en la tabla. 

    ![Gráfico dinámico](media/excel-connector/pivot-table-analyze-pivotchart.png)

1. En el ejemplo siguiente, use **Event Id**, **StartTime** y **EventType** para ver información adicional sobre los eventos meteorológicos.

    ![Visualización de datos](media/excel-connector/visualize-excel-data.png)

1. Cree paneles completos para supervisar los datos.

## <a name="next-steps"></a>Pasos siguientes

[Visualización de datos mediante una consulta de Kusto de Azure Data Explorer importada en Microsoft Excel](excel-blank-query.md)
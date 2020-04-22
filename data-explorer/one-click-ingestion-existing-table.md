---
title: Uso de la ingesta con un clic para ingerir datos en una tabla de Azure Data Explorer existente
description: Ingesta (carga) de datos en una tabla de Azure Data Explore de forma simple, mediante la ingesta con un clic.
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: overview
ms.date: 03/29/2020
ms.openlocfilehash: bd919c43c847ed6e0078752e0b6a0b88edbf68f3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492300"
---
# <a name="use-one-click-ingestion-to-ingest-data-to-an-existing-table-in-azure-data-explorer"></a>Uso de la ingesta con un clic para ingerir datos en una tabla existente en Azure Data Explorer

La ingesta con un clic le permite ingerir rápidamente datos en formato JSON, CSV y otros en una tabla. Con la interfaz de usuario web de Azure Data Explorer, puede ingerir datos del almacenamiento, de un archivo local o de un contenedor. 

En este documento se describe el uso del asistente con un clic intuitivo para ingerir datos JSON de un archivo a una tabla existente. Luego, puede editar la tabla y ejecutar consultas mediante la interfaz de usuario web de Azure Data Explorer.

La ingesta con un clic es especialmente útil cuando se ingieren datos por primera vez, o cuando el esquema de los datos no resulta familiar. 

Para información general sobre la ingesta con un clic y una lista de requisitos previos, consulte [Ingesta con un clic](ingest-data-one-click.md).
Para información sobre la ingesta de datos en una nueva tabla en Azure Data Explorer, consulte [Ingesta con un solo clic en una nueva tabla](one-click-ingestion-new-table.md).

## <a name="ingest-new-data"></a>Ingesta de nuevos datos

1. En el menú izquierdo de la interfaz de usuario web, haga clic con el botón derecho en una *base de datos* o una *tabla* y seleccione **Ingest new data (Preview)** (Ingerir nuevos datos [versión preliminar]).

    ![Seleccione la ingesta con un solo clic en la interfaz de usuario web](media/one-click-ingestion-existing-table/one-click-ingestion-in-webui.png)   
 
1. En la ventana **Ingest new data (Preview)** (Ingerir nuevos datos [versión preliminar]), la pestaña **Source** (Origen) está seleccionada.

1. Si el campo **Table** (Tabla) no se rellena automáticamente, seleccione un nombre de tabla existente del menú de lista desplegable.
    > [!TIP]
    > Si selecciona **Ingest new data (Preview)** [Ingerir datos nuevos (versión preliminar)], en una fila de *tabla*, el nombre de la tabla seleccionada aparecerá en los **Detalles del proyecto**.

[!INCLUDE [data-explorer-one-click-ingestion-types](includes/data-explorer-one-click-ingestion-types.md)]
    
Seleccione **Editar esquema** para ver y editar la configuración de la columna de la tabla.

## <a name="edit-the-schema"></a>Edición del esquema

1. Se abre el cuadro de diálogo **Map columns** (Asignar columnas) y puede asignar columnas de datos de origen a las columnas de la tabla de destino. 
    * En los campos **Source columns** (Columnas de origen), escriba los nombres de columna que se asignarán con los campos **Target columns** (Columnas de destino).
    * Para eliminar una columna de la asignación, seleccione el icono de papelera.

    ![Ventana Asignar columnas](media/one-click-ingestion-existing-table/map-columns.png)

1. Seleccione **Actualizar**.
1. En la pestaña **Schema** (Esquema):
    1. El nombre del archivo de origen seleccionará automáticamente el **tipo de compresión**.

        [!INCLUDE [data-explorer-one-click-ingestion-edit-schema](includes/data-explorer-one-click-ingestion-edit-schema.md)]
        
    1. Si selecciona **JSON**, también debe seleccionar los **niveles JSON**, del 1 al 10. Los niveles afectan a la representación de datos de la columna de tabla.

    ![Selección de los niveles JSON](media/one-click-ingestion-existing-table/json-levels.png)

    * Para formatos tabulares, seleccione la casilla **Include column names** (Incluir nombres de columna) para omitir la fila de encabezado del archivo.
        
    ![Selección de Include column names (Incluir nombres de columna)](media/one-click-ingestion-existing-table/non-json-format.png)

    > [!Note]
    > Los formatos tabulares pueden ingerir datos de columna en una sola columna de una tabla de Azure Data Explorer. 

    * Las nuevas asignaciones se establecen automáticamente, pero puede cambiarlas para usar una existente. 
    * Seleccione el botón **Map columns** (Asignar columnas) para abrir la ventana **Map columns** (Asignar columnas). En la ventana **Map columns** (Asignar columnas) puede asociar una o varias columnas o atributos de origen a las columnas de Azure Data Explorer.

## <a name="copy-and-paste-queries"></a>Copiar y pegar consultas

1. Encima del panel del **Editor**, seleccione el botón **v** para abrir el editor. En el editor, puede ver y copiar los comandos automáticos generados a partir de sus entradas. 
1. En la tabla: 
    * Seleccione nuevos encabezados de columna para agregar valores para **New column** (Nueva columna), **Delete column** (Eliminar columna), **Sort ascending** (Orden ascendente) o **Sort descending** (Orden descendente). En las columnas existentes, solo está disponible la ordenación de datos.

    > [!Note]
    > * No se puede actualizar el nombre y el tipo de datos de las tablas existentes.
    > * Los comandos drop solo revierten los cambios realizados por su flujo de ingesta (nuevas extensiones y columnas). No se quitará ninguna otra cosa.

[![](media/one-click-ingestion-existing-table/edit-view.png "One-click ingestion edit view")](media/one-click-ingestion-existing-table/edit-view.png#lightbox) 

## <a name="start-ingestion"></a>Inicio de la ingesta

Seleccione **Start ingestion** (Iniciar ingesta) para crear una tabla y una asignación, y comenzar la ingesta de datos.

![Inicio de la ingesta](media/one-click-ingestion-existing-table/start-ingestion.png)

## <a name="data-ingestion-completed"></a>Ingesta de datos finalizada

En la ventana **Data ingestion completed** (Ingesta de datos completada), los tres pasos se marcarán con marcas de verificación verdes si la ingesta de datos se completa correctamente.
 
![Ingesta de datos con un solo clic completada](media/one-click-ingestion-existing-table/one-click-data-ingestion-complete.png)

[!INCLUDE [data-explorer-one-click-ingestion-query-data](includes/data-explorer-one-click-ingestion-query-data.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Consulta de datos en la interfaz de usuario web de Azure Data Explorer](/azure/data-explorer/web-query-data)
* [Escritura de consultas para Azure Data Explorer mediante el lenguaje de consulta Kusto](/azure/data-explorer/write-queries)

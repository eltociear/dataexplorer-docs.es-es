---
title: Visualización de datos desde Azure Data Explorer mediante Sisense
description: En este artículo, aprenderá a configurar Azure Data Explorer como un origen de datos para Sisense y a visualizar los datos.
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: conceptual
ms.custom: has-adal-ref
ms.date: 5/29/2019
ms.openlocfilehash: 4afbf9ddf53715b045dba46d03f233f1401ff770
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82862163"
---
# <a name="visualize-data-from-azure-data-explorer-in-sisense"></a>Visualización de datos desde Azure Data Explorer en Sisense

Sisense es una plataforma de inteligencia empresarial de análisis que le permite compilar aplicaciones de análisis que proporcionan experiencias de usuario altamente interactivas. La inteligencia empresarial y el software de informes del panel le permiten acceder a los datos y combinarlos en unos pocos clics. Puede conectarse a orígenes de datos estructurados y no estructurados, combinar tablas desde varios orígenes con un scripting y codificación mínimos, y crear informes y paneles web interactivos. En este artículo, aprenderá a configurar Azure Data Explorer como origen de datos de Sisense y a visualizarlos después desde un clúster de ejemplo.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este artículo, necesitará lo siguiente:

* [Descarga e instalación de la aplicación de Sisense](https://documentation.sisense.com/latest/getting-started/download-install.htm)

* Cree un clúster y una base de datos que incluya los datos de ejemplo de StormEvents. Para más información, consulte [Inicio rápido: Creación de un clúster y de la base de datos de Azure Data Explorer](create-cluster-database-portal.md) e [Ingesta de datos de ejemplo en Azure Data Explorer](ingest-sample-data.md).

    [!INCLUDE [data-explorer-storm-events](includes/data-explorer-storm-events.md)]

## <a name="connect-to-sisense-dashboards-using-azure-data-explorer-jdbc-connector"></a>Conexión a los paneles de Sisense mediante el conector Azure Data Explorer JDBC

1. Descargue y copie las últimas versiones de los siguientes archivos jar a *..\Sisense\DataConnectors\jdbcdrivers\adx*.

    * activation-1.1.jar
    * adal4j-1.6.0.jar
    * commons-codec-1.10.jar
    * commons-collections4-4.1.jar
    * commons-lang3-3.5.jar
    * gson-2.8.0.jar
    * jcip-annotations-1.0-1.jar
    * json-smart-1.3.1.jar
    * lang-tag-1.4.4.jar
    * mail-1.4.7.jar
    * mssql-jdbc-7.2.1.jre8.jar
    * nimbus-jose-jwt-7.0.1.jar
    * oauth2-oidc-sdk-5.24.1.jar
    * slf4j-api-1.7.21.jar

1. Abra la aplicación de **Sisense**.
1. Seleccione la pestaña **Datos** y seleccione **+ElastiCube** para crear un nuevo modelo ElastiCube.

    ![Seleccionar ElastiCube](media/sisense/data-select-elasticube.png)

1. En **Agregar nuevo modelo ElastiCube**, asigne un nombre al modelo ElastiCube y **guarde**.

    ![Agregar nuevo modelo ElastiCube](media/sisense/add-new-elasticube-model.png)

1. Seleccione **+ Datos**.

    ![Seleccionar botón de datos](media/sisense/select-data.png)

1. En la pestaña **Seleccionar conector**, seleccione el conector **JDBC genérico**.

    ![Elegir el conector JDBC](media/sisense/select-connector.png)

1. En la pestaña **Conectar**, complete los siguientes campos para el conector **JDBC genérico** y seleccione **Siguiente**.

    ![Configuración del conector JDBC](media/sisense/jdbc-connector.png)

    |Campo |Descripción |
    |---------|---------|
    |Cadena de conexión     |   `jdbc:sqlserver://<cluster_name.region>.kusto.windows.net:1433;database=<database_name>;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.kusto.windows.net;loginTimeout=30;authentication=ActiveDirectoryPassword`      |
    |Carpeta de archivos JAR de JDBC  |    `..\Sisense\DataConnectors\jdbcdrivers\adx`     |
    |Nombre de clase de controlador    |   `com.microsoft.sqlserver.jdbc.SQLServerDriver`      |
    |Nombre de usuario   |    Nombre de usuario de AAD     |
    |Contraseña     |   Contraseña de usuario de AAD      |

1. En la pestaña **Seleccionar datos**, busque **Seleccionar base de datos** para elegir la base de datos relevante para la que dispone de permisos. En este ejemplo, seleccione *test1*.

    ![seleccionar base de datos](media/sisense/select-database.png)

1. En el panel *test* (nombre de base de datos):
    1. Seleccione el nombre de tabla para obtener una vista previa de la tabla y ver los nombres de columna de la tabla. Puede quitar las columnas innecesarias.
    1. Active la casilla de la tabla relevante para seleccionar esa tabla.
    1. Seleccione **Listo**.

    ![Seleccionar tabla](media/sisense/select-table-see-columns.png)

1. Seleccione **Compilar** para generar el conjunto de datos.

    * En la ventana de **compilación**, seleccione **Compilar**.

      ![Ventana de compilación](media/sisense/build-window.png)

    * Espere hasta que el proceso de compilación se complete y, a continuación, seleccione **Compilación correcta**.

      ![Compilación correcta](media/sisense/build-succeeded.png)

## <a name="create-sisense-dashboards"></a>Creación de paneles de Sisense

1. En la pestaña **Análisis**, seleccione **+**  > **Nuevo panel** para crear paneles en este conjunto de datos.

    ![Nuevo panel](media/sisense/new-dashboard.png)

1. Elija un panel y seleccione **Crear**.

    ![Crear panel](media/sisense/create-dashboard.png)

1. En **Nuevo widget**, seleccione **+ Seleccionar datos** para crear un nuevo widget.

    ![Adición de campos al panel de StormEvents](media/sisense/storm-dashboard-add-field.png)

1. Seleccione **+ Agregar más datos** para añadir columnas adicionales a su grafo.

    ![Adición de más datos al grafo](media/sisense/add-more-data.png)

1. Seleccione **+ Widget** para crear otro widget. Arrastre y suelte widgets para reorganizar su panel.

    ![Panel de Storm](media/sisense/final-dashboard.png)

Ahora puede explorar los datos con análisis visuales, crear paneles adicionales y transformar datos en conclusiones accionables para crear un impacto en su negocio.

## <a name="next-steps"></a>Pasos siguientes

* [Write queries for Azure Data Explorer](write-queries.md) (Escritura de consultas del Explorador de datos de Azure)

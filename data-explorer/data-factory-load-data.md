---
title: Copia de datos de Azure Data Factory a Azure Data Explorer
description: En este artículo aprenderá a ingerir (cargar) datos en Azure Data Explorer mediante la herramienta de copia de Azure Data Factory.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/15/2019
ms.openlocfilehash: c937d69df73ed523d1e4f3f0614eadf249ced0f3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493156"
---
# <a name="copy-data-to-azure-data-explorer-by-using-azure-data-factory"></a>Copia de datos en Azure Data Explorer mediante Azure Data Factory 

Azure Data Explorer es un servicio de análisis de datos rápido y totalmente administrado. Ofrece análisis en tiempo real de grandes volúmenes de datos que se transmiten desde muchos orígenes, como aplicaciones, sitios web y dispositivos IoT. Con Azure Data Explorer, puede explorar datos de forma iterativa e identificar patrones y anomalías para mejorar los productos y la experiencia del cliente, supervisar los dispositivos e impulsar las operaciones. Le ayuda a explorar nuevas preguntas y obtener respuestas en cuestión de minutos. 

Azure Data Factory es un servicio de integración de datos basado en la nube totalmente administrado. Se puede usar para rellenar la base de datos de Azure Data Explorer con datos del sistema existente. Puede ayudarle a ahorrar tiempo al compilar soluciones de análisis.

Al cargar datos en Azure Data Explorer, Data Factory ofrece las siguientes ventajas:

* **Instalación sencilla**: obtenga un asistente intuitivo en cinco pasos sin necesidad de scripting.
* **Amplia compatibilidad para el almacenamiento de datos**: obtenga compatibilidad integrada para un amplio conjunto de almacenes de datos tanto locales como en la nube. Para una lista detallada, consulte la tabla de [almacenes de datos admitidos](/azure/data-factory/copy-activity-overview#supported-data-stores-and-formats).
* **Seguro y compatible**: los datos se transfieren a través de HTTPS o Azure ExpressRoute. La presencia del servicio global garantiza que los datos nunca abandonan el límite geográfico.
* **Alto rendimiento**: la velocidad de carga de datos en Azure Data Explorer es de hasta 1 gigabyte por segundo (GBps). Para obtener más información, vea el artículo [Copiar rendimiento de actividad](/azure/data-factory/copy-activity-performance).

En este artículo se utiliza la herramienta Copiar datos de Data Factory para cargar datos de Amazon Simple Storage Service (S3) en Azure Data Explorer. Puede seguir el mismo proceso para copiar datos de otros tipos de almacenes de datos, tales como:
* [Almacenamiento de blobs de Azure](/azure/data-factory/connector-azure-blob-storage)
* [Azure SQL Database](/azure/data-factory/connector-azure-sql-database)
* [Azure SQL Data Warehouse](/azure/data-factory/connector-azure-sql-data-warehouse)
* [Google BigQuery](/azure/data-factory/connector-google-bigquery)
* [Oracle](/azure/data-factory/connector-oracle)
* [Sistema de archivos](/azure/data-factory/connector-file-system)

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Un clúster y la base de datos de Azure Data Explorer](create-cluster-database-portal.md).
* Un origen de datos.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Inicie sesión en [Azure Portal](https://ms.portal.azure.com).

1. En el panel izquierdo, seleccione **Crear un recurso** > **Analytics** > **Data Factory**.

   ![Creación de una factoría de datos en Azure Portal](media/data-factory-load-data/create-adf.png)

1. En el panel **New data factory** (Nueva factoría de datos), especifique valores para los campos de la tabla siguiente:

   ![Panel de nueva factoría de datos](media/data-factory-load-data/my-new-data-factory.png)  

   | Configuración  | Valor que se va a especificar  |
   |---|---|
   | **Nombre** | En el cuadro, escriba un nombre único global para la factoría de datos. Si recibe el error *El nombre de la factoría de datos \"LoadADXDemo\" no está disponible*, escriba otro distinto. Para conocer las reglas sobre nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](/azure/data-factory/naming-rules).|
   | **Suscripción** | En la lista desplegable, seleccione la suscripción de Azure en la que quiere crear la factoría de datos. |
   | **Grupo de recursos** | Seleccione **Crear nuevo** y escriba el nombre de un nuevo grupo de recursos. Si ya tiene un grupo de recursos, seleccione **Usar existente**. |
   | **Versión** | En la lista desplegable, seleccione **V2**. |    
   | **Ubicación** | En la lista desplegable, seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones o regiones. |

1. Seleccione **Crear**.

1. Para supervisar el proceso de creación, seleccione **Notificaciones** en la barra de herramientas. Una vez creada la factoría de datos, selecciónela.
   
   Se abre el panel **Data Factory**.

   ![Panel Data Factory](media/data-factory-load-data/data-factory-home-page.png)

1. Para abrir la aplicación en otro panel, seleccione el icono **Author & Monitor** (Crear y supervisar).

## <a name="load-data-into-azure-data-explorer"></a>Carga de datos en Azure Data Explorer

Puede cargar datos procedentes de muchos tipos de [almacenes de datos](/azure/data-factory/copy-activity-overview#supported-data-stores-and-formats) en Azure Data Explorer. En este artículo se describe cómo cargar datos de Amazon S3.

Puede cargar los datos de una de las maneras siguientes:

* En la interfaz de usuario de Azure Data Factory, en el panel izquierdo, seleccione el icono **Crear**. Esto se muestra en la sección "Crear una factoría de datos" de [Creación de una factoría de datos con la interfaz de usuario de Azure Data Factory](/azure/data-factory/quickstart-create-data-factory-portal#create-a-data-factory).
* En la herramienta Copiar datos de Azure Data Factory, tal como se muestra en [Uso de la herramienta Copiar datos para copiar datos](/azure/data-factory/quickstart-create-data-factory-copy-data-tool).

### <a name="copy-data-from-amazon-s3-source"></a>Copia de datos desde Amazon S3 (origen)

1. En el panel **Let's get started** (Introducción), seleccione **Copy Data** (Copiar datos) para abrir la herramienta Copiar datos.

   ![Botón de la herramienta Copiar datos](media/data-factory-load-data/copy-data-tool-tile.png)

1. En el cuadro **Task name** (Nombre de tarea) del panel **Properties** (Propiedades), escriba un nombre y seleccione **Next** (Siguiente).

    ![Panel de propiedades de Copiar datos](media/data-factory-load-data/copy-from-source.png)

1. En el panel **Source data store** (Almacén de datos de origen), seleccione **Crear nueva conexión**.

    ![Panel de almacén de datos de origen de Copiar datos](media/data-factory-load-data/source-create-connection.png)

1. Seleccione **Amazon S3** y luego, **Continuar**.

    ![Panel de nuevo servicio vinculado](media/data-factory-load-data/amazons3-select-new-linked-service.png)

1. En el panel **New Linked Service (Amazon S3)** (Nuevo servicio vinculado [Amazon S3]), haga lo siguiente:

    ![Especificación del servicio vinculado de Amazon S3](media/data-factory-load-data/amazons3-new-linked-service-properties.png)

    a. En el cuadro **Name** (Nombre), escriba el nombre del nuevo servicio vinculado.

    b. En la lista desplegable **Connect via integration runtime** (Conectar mediante IR), seleccione el valor.

    c. En el cuadro **Access Key ID** (Identificador de clave de acceso), escriba el valor.
    
    > [!NOTE]
    > En Amazon S3, para buscar la clave de acceso, seleccione su nombre de usuario de Amazon en la barra de navegación y luego, **My Security Credentials** (Mis credenciales de seguridad).
    
    d. En el cuadro **Secret Access Key** (Clave de acceso secreta), escriba un valor.

    e. Para probar la conexión de servicio vinculado que creó, seleccione **Probar conexión**.

    f. Seleccione **Finalizar**.
    
      En el panel **Source data store** (Almacén de datos de origen) se muestra la nueva conexión AmazonS31. 

1. Seleccione **Next** (Siguiente).

   ![Conexión creada mediante almacén de datos de origen](media/data-factory-load-data/source-data-store-created-connection.png)

1. En el panel **Seleccione el archivo o la carpeta de entrada**, haga lo siguiente:

    a. Busque el archivo o la carpeta que quiere copiar y selecciónelo.

    b. Seleccione el comportamiento de copia que quiera. Asegúrese de que la casilla **Binary Copy** (Copia binaria) esté desactivada.

    c. Seleccione **Next** (Siguiente).

    ![Elegir archivo o carpeta de entrada](media/data-factory-load-data/source-choose-input-file.png)

1. En el panel **File format settings** (Configuración de formato de archivo), seleccione la configuración relevante para su archivo. Luego, seleccione **Next** (Siguiente).

   ![Panel de configuración de formato de archivo](media/data-factory-load-data/source-file-format-settings.png)

### <a name="copy-data-into-azure-data-explorer-destination"></a>Copia de datos en Azure Data Explorer (destino)

Se ha creado el servicio vinculado de Azure Data Explorer para copiar los datos en la tabla de destino de Azure Data Explorer (receptor) que se especifica en esta sección.

> [!NOTE]
> Use la [actividad de comando de Azure Data Factory para ejecutar comandos de control de Azure Data Explorer](data-factory-command-activity.md) y utilice cualquiera de los [comandos de ingesta desde consulta](kusto/management/data-ingestion/ingest-from-query.md), como `.set-or-replace`.

#### <a name="create-the-azure-data-explorer-linked-service"></a>Creación del servicio vinculado de Azure Data Explorer

Para crear el servicio vinculado de Azure Data Explorer, haga lo siguiente:

1. Para usar una conexión del almacén de datos existente o especificar un nuevo almacén de datos, en el panel **Destination data store** (Almacén de datos de destino), seleccione **Create new connection** (Crear conexión).

    ![Panel de almacén de datos de destino](media/data-factory-load-data/destination-create-connection.png)

1. En el panel **New Linked Service** (Nuevo servicio vinculado), seleccione **Azure Data Explorer** y después **Continuar**.

    ![Panel de nuevo servicio vinculado](media/data-factory-load-data/adx-select-new-linked-service.png)

1. En el panel **New Linked Service (Azure Data Explorer)** (Nuevo servicio vinculado [Azure Data Explorer]), haga lo siguiente:

    ![Panel de nuevo servicio vinculado de Azure Data Explorer](media/data-factory-load-data/adx-new-linked-service.png)

   a. En el cuadro **Name** (Nombre), escriba un nombre para el servicio vinculado de Azure Data Explorer.

   b. En **Método de selección de cuentas**, elija una de las siguientes opciones: 

    * Seleccione **From Azure subscription** (Desde suscripción de Azure) y luego, en las listas desplegables, seleccione su **suscripción de Azure** y su **clúster**. 

        > [!NOTE]
        > En el control desplegable **Cluster** (Clúster) solo aparecen los clústeres asociados a la suscripción.

    * Seleccione **Enter manually** (Especificar manualmente) y especifique su **punto de conexión**.

   c. En el cuadro **Inquilino**, escriba el nombre del inquilino.

   d. En el cuadro **Id. de entidad de servicio**, escriba el identificador de la entidad de servicio.

   e. Seleccione **Clave de entidad de servicio** y luego, en el cuadro **Clave de entidad de servicio**, escriba el valor de la clave.

   f. En la lista desplegable **Base de datos**, seleccione el nombre de la base de datos. También puede activar la casilla **Editar** y escribir el nombre de la base de datos.

   g. Para probar la conexión de servicio vinculado que creó, seleccione **Probar conexión**. Si puede conectarse al servicio vinculado, el panel muestra una marca de verificación verde y un mensaje **Conexión correcta**.

   h. Seleccione **Finalizar** para completar la creación del servicio vinculado.

    > [!NOTE]
    > Azure Data Factory usa la entidad de servicio para acceder al servicio de Azure Data Explorer. Para crear una entidad de servicio, vaya a [Cree una entidad de servicio de Azure Active Directory (Azure AD)](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-service-principal). No utilice el método de Azure Key Vault.

#### <a name="configure-the-azure-data-explorer-data-connection"></a>Configuración de la conexión de datos de Azure Data Explorer

Una vez creada la conexión de servicio vinculado, se abre el panel **Destination data store** (Almacén de datos de destino) y la conexión que creó está disponible para su uso. Para configurar la conexión, haga lo siguiente:

1. Seleccione **Next** (Siguiente).

    ![Panel de almacén de datos de destino de Azure Data Explorer](media/data-factory-load-data/destination-data-store.png)

1. Enel panel **Table mapping** (Asignación de tabla), establezca el nombre de tabla de destino y seleccione **Next** (Siguiente).

    ![Panel de asignación de tabla del conjunto de datos de destino](media/data-factory-load-data/destination-dataset-table-mapping.png)

1. En el panel **Asignación de columnas**, se realizan las siguientes asignaciones:

    a. Azure Data Factory realiza la primera asignación según la [asignación de esquemas de Azure Data Factory](/azure/data-factory/copy-activity-schema-and-type-mapping). Haga lo siguiente:

    * Establezca las **asignaciones de columna** para la tabla de destino de Azure Data Factory. La asignación predeterminada se muestra desde el origen hasta la tabla de destino de Azure Data Factory.

    * Cancele la selección de las columnas que no necesite para definir su asignación de columnas.

    b. La segunda asignación se produce cuando estos datos tabulares se insertan en Azure Data Explorer. La asignación se realiza según las [reglas de asignación de CSV](kusto/management/mappings.md#csv-mapping). Aunque los datos de origen no estén en formato CSV, Azure Data Factory convierte los datos a un formato tabular. Por lo tanto, la asignación de CSV es la única asignación relevante en esta fase. Haga lo siguiente:

    * (Opcional) En las **propiedades del receptor de Azure Data Explorer (Kusto)** , agregue el **nombre de asignación de ingesta** de manera que se pueda usar la asignación de columnas.

    * Si no se especifica el **nombre de asignación de ingesta**, se usará el orden de asignación *por nombre* en la sección **Asignaciones de columnas**. Si se produce un error en la asignación *por nombre*, Azure Data Explorer intenta ingerir los datos en el orden de *posición por columna* (es decir, se asigna por posición de forma predeterminada).

    * Seleccione **Next** (Siguiente).

    ![Panel de asignación de columnas del conjunto de datos de destino](media/data-factory-load-data/destination-dataset-column-mapping.png)

1. En el panel **Configuración**, haga lo siguiente:

    a. En **Fault tolerance settings** (Configuración de tolerancia a errores), escriba la configuración correspondiente.

    b. En **Performance settings** (Configuración de rendimiento), no se aplica **Enable staging** (Permitir almacenamiento provisional) y **Advanced settings** (Configuración avanzada) incluye consideraciones de costo. Si no tiene ningún requisito específico, deje esta configuración tal y como está.

    c. Seleccione **Next** (Siguiente).

    ![Panel de configuración de Copiar datos](media/data-factory-load-data/copy-data-settings.png)

1. En el panel **Summary** (Resumen), revise la configuración y seleccione **Next** (Siguiente).

    ![Panel de resumen de Copiar datos](media/data-factory-load-data/copy-data-summary.png)

1. En el panel **Deployment complete** (Implementación finalizada), haga lo siguiente:

    a. Para pasar a la pestaña **Supervisar** y ver el estado de la canalización (es decir, progreso, errores y flujo de datos), seleccione **Supervisar**.

    b. Para poder editar los servicios vinculados, los conjuntos de datos y las canalizaciones, seleccione **Editar canalización**.

    c. Seleccione **Finalizar** para completar la tarea de copia de datos.

    ![Panel de implementación finalizada](media/data-factory-load-data/deployment.png)

## <a name="next-steps"></a>Pasos siguientes

* Puede obtener más información sobre [Azure Data Explorer](/azure/data-factory/connector-azure-data-explorer) en Azure Data Factory.
* Puede obtener más información sobre la edición de servicios vinculados, conjuntos de datos y canalizaciones en la [interfaz de usuario de Data Factory](/azure/data-factory/quickstart-create-data-factory-portal).
* Obtenga más información sobre las [consultas de Azure Data Explorer](/azure/data-explorer/web-query-data) para las consultas de datos.

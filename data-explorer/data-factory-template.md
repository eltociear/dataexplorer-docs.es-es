---
title: Copia en bloque desde una base de datos a Azure Data Explorer mediante la plantilla de Azure Data Factory
description: En este artículo aprenderá a usar una plantilla de Azure Data Factory para copiar en bloque desde una base de datos a Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/08/2019
ms.openlocfilehash: e1c51132ddb36a206a97236b6ceb4553a62e6d2c
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373912"
---
# <a name="copy-in-bulk-from-a-database-to-azure-data-explorer-by-using-the-azure-data-factory-template"></a>Copia en bloque desde una base de datos a Azure Data Explorer mediante la plantilla de Azure Data Factory 

Azure Data Explorer es un servicio de análisis de datos rápido y totalmente administrado. Ofrece análisis en tiempo real de grandes volúmenes de datos que se transmiten desde muchos orígenes, como aplicaciones, sitios web y dispositivos IoT. 

Para copiar datos de una base de datos de Oracle Server, Netezza, Teradata o SQL Server en Azure Data Explorer, tiene que cargar enormes cantidades de datos de varias tablas. Normalmente, los datos se tienen que dividir aún más en cada tabla para poder cargar filas con varios subprocesos en paralelo desde una única tabla. En este artículo se describe una plantilla para utilizarla en estos escenarios.

Las [plantillas de Azure Data Factory](/azure/data-factory/solution-templates-introduction) son canalizaciones de Data Factory predefinidas. Estas plantillas pueden ayudarle a empezar a trabajar rápidamente con Data Factory y a reducir el tiempo de desarrollo necesario en los proyectos de integración de datos. 

La plantilla de *copia masiva desde la base de datos a Azure Data Explorer* se crea mediante las actividades *Lookup* y *ForEach*. Si quiere copiar los datos con más rapidez, puede usar la plantilla para crear muchas canalizaciones por base de datos o por tabla. 

> [!IMPORTANT]
> Asegúrese de usar la herramienta adecuada para la cantidad de datos que desea copiar.
> * Use la plantilla de *copia masiva desde la base de datos a Azure Data Explorer* para copiar grandes cantidades de datos desde bases de datos como SQL Server y Google BigQuery a Azure Data Explorer. 
> * Use la [*herramienta Copiar datos de Data Factory*](data-factory-load-data.md) para copiar algunas tablas con cantidades pequeñas o moderadas de datos a Azure Data Explorer. 

## <a name="prerequisites"></a>Prerequisites

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Un clúster y la base de datos de Azure Data Explorer](create-cluster-database-portal.md).
* [Debe crear una factoría de datos](data-factory-load-data.md#create-a-data-factory).
* Un origen de datos en una base de datos.

## <a name="create-controltabledataset"></a>Creación de ControlTableDataset

*ControlTableDataset* indica qué datos se copiarán del origen al destino en la canalización. El número de filas indica el número total de canalizaciones necesarias para copiar los datos. ControlTableDataset debe definirse como parte de la base de datos de origen.

En el código siguiente se muestra un ejemplo del formato de la tabla de origen de SQL Server:
    
```sql   
CREATE TABLE control_table (
PartitionId int,
SourceQuery varchar(255),
ADXTableName varchar(255)
);
```

Los elementos de código se describen en la tabla siguiente:

|Propiedad  |Descripción  | Ejemplo
|---------|---------| ---------|
|PartitionId   |  Orden de copia | 1  |  
|SourceQuery   |  Consulta que indica qué datos se copiarán durante el tiempo de ejecución de la canalización. | <br>`select * from table where lastmodifiedtime  LastModifytime >= ''2015-01-01 00:00:00''>` </br>    
|ADXTableName  |  Nombre de la tabla de destino. | MyAdxTable       |  

Si su elemento ControlTableDataset tiene un formato diferente, al crearlo adáptelo para su formato.

## <a name="use-the-bulk-copy-from-database-to-azure-data-explorer-template"></a>Uso de la plantilla de copia masiva de la base de datos a Azure Data Explorer

1. En el panel **Let's get started** (Introducción), seleccione **Create pipeline from template** (Creación de una canalización a partir de una plantilla) para abrir el panel **Template gallery** (Galería de plantillas).

    ![Panel "Let's get started" (Introducción) de Azure Data Factory](media/data-factory-template/adf-get-started.png)

1. Seleccione la plantilla **Bulk Copy from Database to Azure Data Explorer**.
 
    ![Plantilla "Bulk Copy from Database to Azure Data Explorer" (Copia masiva desde la base de datos a Azure Data Explorer)](media/data-factory-template/pipeline-from-template.png)

1.  En el panel **Bulk Copy from Database to Azure Data Explorer** (Copia masiva desde la base de datos a Azure Data Explorer), en **User Inputs** (Entradas de usuario), especifique los conjuntos de datos del siguiente modo: 

    a. En la lista desplegable **ControlTableDataset**, seleccione el servicio vinculado a la tabla de control que indica qué datos se copian del origen al destino y dónde se colocarán en el destino. 

    b. En la lista desplegable **SourceDataset**, seleccione el servicio vinculado a la base de datos de origen. 

    c. En la lista desplegable **AzureDataExplorerTable**, seleccione la tabla de Azure Data Explorer. Si el conjunto de datos no existe, [cree el servicio vinculado de Azure Data Explorer](data-factory-load-data.md#create-the-azure-data-explorer-linked-service) para agregar el conjunto de datos.

    d. Seleccione **Usar esta plantilla**.

    ![Panel "Bulk Copy from Database to Azure Data Explorer" (Copia masiva desde la base de datos a Azure Data Explorer)](media/data-factory-template/configure-bulk-copy-adx-template.png)

1. Seleccione un área del lienzo (fuera de las actividades) para acceder a la canalización de la plantilla. Seleccione la pestaña **Parameters** (Parámetros) para especificar los parámetros de la tabla, incluidos los valores de **Name** (Nombre) (nombre de la tabla de control) y **Default value** (Valor predeterminado) (nombres de columna).

    ![Parámetros de canalización](media/data-factory-template/pipeline-parameters.png)

1.  En **Lookup**, seleccione **GetPartitionList** para ver la configuración predeterminada. Automáticamente se crea una consulta.
1.  Seleccione la actividad Command, **ForEachPartition**, seleccione la pestaña **Settings** (Configuración) y, a continuación, haga lo siguiente:

    a. En el cuadro **Batch count** (Número de lotes), escriba un número entre 1 y 50. Esta selección determina el número de canalizaciones que se ejecutan en paralelo hasta que se alcanza el número de filas de *ControlTableDataset*. 

    b. Para asegurarse de que los lotes de canalización se ejecutan en paralelo, *no* active la casilla **Sequential** (Secuencial).

    ![Configuración de ForEachPartition](media/data-factory-template/foreach-partition-settings.png)

    > [!TIP]
    > El procedimiento recomendado consiste en ejecutar muchas canalizaciones en paralelo, para que los datos se puedan copiar más rápidamente. Para aumentar la eficacia, particione los datos en la tabla de origen y asigne una partición por canalización, según la fecha y la tabla.

1. Seleccione **Validate All** (Validar todo) para validar la canalización de Azure Data Factory y consulte el resultado en el panel **Pipeline Validation Output** (Salida de validación de canalización).

    ![Validar las canalizaciones de la plantilla](media/data-factory-template/validate-template-pipelines.png)

1. Si es necesario, seleccione **Debug** (Depurar) y, a continuación, seleccione **Add trigger** (Agregar desencadenador) para ejecutar la canalización.

    ![Botones "Debug" (Depurar) y "Add trigger" (Agregar desencadenador)](media/data-factory-template/trigger-run-of-pipeline.png)    

Ahora puede usar la plantilla para copiar eficazmente grandes cantidades de datos de las bases de datos y las tablas.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [copiar datos en Azure Data Explorer mediante Azure Data Factory](data-factory-load-data.md).
* Puede obtener más información sobre [Azure Data Explorer](/azure/data-factory/connector-azure-data-explorer) en Azure Data Factory.
* Obtenga más información sobre las [consultas de Azure Data Explorer](web-query-data.md) para las consultas de datos.







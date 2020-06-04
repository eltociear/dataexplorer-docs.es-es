---
title: Introducción a la ingesta de datos en Azure Data Explorer
description: Información acerca de las diferentes maneras que puede usar para la ingesta (carga) de datos en Azure Data Explorer
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/18/2020
ms.openlocfilehash: 6fa60c3c82a889d1161b30529586b225cee3efbd
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863289"
---
# <a name="azure-data-explorer-data-ingestion-overview"></a>Introducción a la ingesta de datos en Azure Data Explorer 

La ingesta de datos es el proceso que se usa para cargar los registros de datos de uno o varios orígenes para importar datos en una tabla en Azure Data Explorer. Una vez ingeridos, los datos están disponibles para su consulta.

El diagrama siguiente muestra el flujo de un extremo a otro para trabajar en Azure Data Explorer y muestra diferentes métodos de ingesta.

:::image type="content" source="media/data-ingestion-overview/data-management-and-ingestion-overview.png" alt-text="Introducción al esquema de administración e ingesta de datos":::

El servicio de administración de datos Azure Data Explorer, que es el responsable de la ingesta de datos, implementa el siguiente proceso:

Azure Data Explorer extrae los datos de un origen externo y lee las solicitudes de una cola de pendientes de Azure. Los datos se procesan por lotes o se transmiten a Data Manager. El procesamiento por lotes de los datos que fluyen en la misma base de datos y tabla se optimiza para mejorar el rendimiento de la ingesta. Azure Data Explorer valida los datos iniciales y convierte los formatos de datos cuando es necesario. Una posterior manipulación de los datos incluye hacer coincidir los esquemas, así como organizar, indexar, codificar y comprimir los datos. Los datos se conservan en el almacenamiento de acuerdo con la directiva de retención establecida. A continuación, Data Manager confirma la ingesta de datos en el motor, donde están disponibles para su consulta. 

## <a name="supported-data-formats-properties-and-permissions"></a>Formatos de datos compatibles, propiedades y permisos

* **[Formatos de datos compatibles](ingestion-supported-formats.md)** 

* **[Propiedades de la ingesta](ingestion-properties.md)** : las propiedades que afectan a la forma en que se van a ingerir los datos (por ejemplo, etiquetado, asignación u hora de creación).

* **Permisos**: Para ingerir datos, el proceso necesita[permisos de nivel de agente de ingesta de bases de datos](kusto/management/access-control/role-based-authorization.md). Otras acciones, como la consulta, pueden requerir permisos de administrador de base de datos, usuario de base de datos o administrador de tabla.


## <a name="batching-vs-streaming-ingestion"></a>Ingesta de procesamiento por lotes frente a ingesta de streaming

* La ingesta de procesamiento por lotes realiza el procesamiento por lotes de los datos y está optimizada para lograr un alto rendimiento de la ingesta. Este método es el tipo de ingesta preferido y de mayor rendimiento. Los datos se procesan por lotes en función de las propiedades de la ingesta. Después se combinan y optimizan pequeños lotes de datos para agilizar los resultados de la consulta. La directiva de [procesamiento por lotes de la ingesta](kusto/management/batchingpolicy.md) se puede establecer en bases de datos o en tablas. De forma predeterminada, el valor máximo del procesamiento por lotes es 5 minutos, 1000 elementos o un tamaño total de 500 MB.

* [Ingesta en streaming](ingest-data-streaming.md) es la ingesta de datos en curso desde un origen de streaming. La ingesta de streaming permite una latencia casi en tiempo real para pequeños conjuntos pequeños de datos por tabla. En un principio, los datos se ingieren en el almacén de filas y posteriormente se mueven a las extensiones del almacén de columnas. La ingesta en streaming se puede realizar mediante una biblioteca de cliente de Azure Data Explorer, o bien desde una de las canalizaciones de datos admitidas. 

## <a name="ingestion-methods-and-tools"></a>Métodos y herramientas de ingesta

Azure Data Explorer admite varios métodos de ingesta, cada uno con sus propios escenarios de destino. Estos métodos incluyen herramientas de ingesta, conectores y complementos para diversos servicios, canalizaciones administradas, ingesta mediante programación mediante distintos SDK y acceso directo a la ingesta.

### <a name="ingestion-using-managed-pipelines"></a>Ingesta mediante canalizaciones administradas

Para aquellas organizaciones que deseen que sea un servicio externo el que realice la administración (límites, reintentos, supervisiones, alertas, etc.), es probable que un conector sea la solución más adecuada. La ingesta en cola es apropiada para grandes volúmenes de datos. Azure Data Explorer admite las siguientes instancias de Azure Pipelines:

* **[Event Grid](https://azure.microsoft.com/services/event-grid/)** : una canalización que escucha Azure Storage y actualiza Azure Data Explorer para extraer información cuando se producen eventos suscritos. Para más información, consulte [Ingesta de blobs de Azure en Azure Data Explorer](ingest-data-event-grid.md).

* **[Event Hub](https://azure.microsoft.com/services/event-hubs/)** : una canalización que transfiere eventos de los servicios a Azure Data Explorer. Para más información, consulte [Ingesta de datos desde el centro de eventos en Azure Data Explorer](ingest-data-event-hub.md).

* **[IoT Hub](https://azure.microsoft.com/services/iot-hub/)** : una canalización que se usa para la transferencia de datos desde dispositivos IoT compatibles a Azure Data Explorer. Para más información, consulte [Ingesta de IoT Hub](ingest-data-iot-hub.md).

### <a name="ingestion-using-connectors-and-plugins"></a>Ingesta mediante conectores y complementos

* **Complemento Logstash**, consulte [Ingesta de datos de Logstash en Azure Data Explorer](ingest-data-logstash.md).

* **Conector de Kafka**, consulte [Ingesta de datos de Kafka en Azure Data Explorer](ingest-data-kafka.md).

* **[Power Automate](https://flow.microsoft.com/)** : una canalización de flujos de trabajo automatizada a Azure Data Explorer. Power Automate se puede usar para ejecutar una consulta y realizar acciones preestablecidas con los resultados de la consulta como desencadenador. Consulte [Conector de Azure Data Explorer para Power Automate (versión preliminar)](flow.md).

* **Conector de Apache Spark**:  proyecto de código abierto que se puede ejecutar en cualquier clúster de Spark. Implementa el origen y el receptor de datos para mover datos entre los clústeres de Azure Data Explorer y de Spark. Puede compilar aplicaciones rápidas y escalables orientadas a escenarios controlados por datos. Consulte [Conector de Azure Data Explorer para Apache Spark](spark-connector.md).

### <a name="programmatic-ingestion-using-sdks"></a>Ingesta mediante programación mediante SDK

Azure Data Explorer proporciona SDK que pueden usarse para la consulta e ingesta de datos. La ingesta mediante programación está optimizada para reducir los costos de ingesta (COG), minimizando las transacciones de almacenamiento durante y después del proceso de ingesta.

**SDK y proyectos de código abierto disponibles**

* [SDK de Python](kusto/api/python/kusto-python-client-library.md)

* [SDK de .NET](kusto/api/netfx/about-the-sdk.md)

* [SDK de Java](kusto/api/java/kusto-java-client-library.md)

* [SDK de Node](kusto/api/node/kusto-node-client-library.md)

* [REST API](kusto/api/netfx/kusto-ingest-client-rest.md)

* [GO API](kusto/api/golang/kusto-golang-client-library.md)

### <a name="tools"></a>Herramientas

* **Azure Data Factory (ADF)** : un servicio de integración de datos totalmente administrado para cargas de trabajo de análisis en Azure. Azure Data Factory se conecta con más de 90 orígenes admitidos para proporcionar una transferencia de datos eficaz y resistente. ADF prepara, transforma y enriquece los datos para proporcionar información que se puede supervisar de varias formas. Este servicio se puede usar como solución de un solo uso, en una escala de tiempo periódica o desencadenada por eventos específicos. 
  * [Integración de Azure Data Explorer con Azure Data Factory](data-factory-integration.md).
  * [Uso de Azure Data Factory para copiar datos de orígenes compatibles a Azure Data Explorer](/azure/data-explorer/data-factory-load-data).
  * [Copia en bloque desde una base de datos a Azure Data Explorer mediante la plantilla de Azure Data Factory](data-factory-template.md).
  * [Uso de la actividad de comandos de Azure Data Factory para ejecutar comandos de control de Azure Data Explorer](data-factory-command-activity.md).

* **[Ingesta con un solo clic](ingest-data-one-click.md)** : Permite ingerir datos rápidamente mediante la creación y ajuste de tablas a partir de una amplia gama de tipos de origen. La ingesta con un solo clic sugiere tablas y estructuras de asignación automáticamente en función del origen de datos de Azure Data Explorer. La ingesta con un solo clic se puede usar para la ingesta puntual, o bien para definir una ingesta continua a través de Event Grid en el contenedor en el que se han ingerido los datos.

* **[LightIngest](lightingest.md)** : utilidad de línea de comandos para la ingesta de datos ad-hoc en Azure Data Explorer. La utilidad puede extraer datos de origen de una carpeta local o de un contenedor de almacenamiento de blobs de Azure.

### <a name="kusto-query-language-ingest-control-commands"></a>Comandos de control de ingesta del lenguaje de consulta de Kusto

Hay varios métodos por los que los datos se pueden ingerir directamente al motor mediante los comandos del lenguaje de consulta de Kusto (KQL). Dado que este método omite los servicios de Administración de datos, solo es adecuado para la exploración y la creación de prototipos. No se debe usar en escenarios de producción o de gran volumen.

  * **Ingesta insertada**:  se envía un comando de control [.ingest inline](kusto/management/data-ingestion/ingest-inline.md) al motor y los datos que se van a ingerir forman parte del propio texto del comando. Este método está pensado para la realización de pruebas improvisadas.

  * **Ingesta desde consulta**: se envía un comando de control [.set, .append, .set-or-append o .set-or-replace](kusto/management/data-ingestion/ingest-from-query.md) al motor y los datos se especifican indirectamente como los resultados de una consulta o un comando.

  * **Ingesta desde almacenamiento (extracción)** : se envía un comando de control [.ingest into](kusto/management/data-ingestion/ingest-from-storage.md) al motor con los datos almacenados en algún almacenamiento externo (por ejemplo, Azure Blob Storage) al que el motor puede acceder y al que el comando señala.

## <a name="comparing-ingestion-methods-and-tools"></a>Comparación de métodos y herramientas de ingesta

| Nombre de ingesta | Tipo de datos | Tamaño de archivo máximo | Streaming, procesamiento por lotes, directo | La mayoría de los escenarios comunes | Consideraciones |
| --- | --- | --- | --- | --- | --- |
| [**Ingesta con un solo clic**](ingest-data-one-click.md) | *sv, JSON | 1 GB sin comprimir (vea la nota)| Procesamiento por lotes en el contenedor, el archivo local y el blob en la ingesta directa. | Un esquema de creación de tablas de un solo uso, definición de ingesta continua con Event Grid, ingesta en bloque con contenedor (hasta 10 000 blobs). | Se seleccionan aleatoriamente 10 000 del contenedor.|
| [**LightIngest**](lightingest.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes a través del DM o de la ingesta directa al motor. |  Migración de datos, datos históricos con marcas de tiempo de ingesta ajustadas, ingesta en bloque (sin restricción de tamaño).| Distingue mayúsculas de minúsculas, con distinción de espacio. |
| [**ADX Kafka**](ingest-data-kafka.md) | | | | |
| [**ADX a Apache Spark**](spark-connector.md) | | | | |
| [**LogStash**](ingest-data-logstash.md) | | | | |
| [**Azure Data Factory**](kusto/tools/azure-data-factory.md) | [Formatos de datos compatibles](/azure/data-factory/copy-activity-overview#supported-data-stores-and-formats) | Ilimitado * (por restricciones de ADF) | Procesamiento por lotes o por desencadenador de Azure Data Factory. | Admite formatos que normalmente no se admiten, archivos grandes, puede copiar de más de 90 orígenes, desde permanentes hasta la nube. | Hora de la ingesta |
|[ **Azure Data Flow**](kusto/tools/flow.md) | | | | Comandos de ingesta como parte del flujo.| Debe tener un tiempo de respuesta de alto rendimiento. |
| [**IoT Hub**](kusto/management/data-ingestion/iothub.md) | [Formatos de datos compatibles](kusto/management/data-ingestion/iothub.md#data-format)  | N/D | Procesamiento por lotes, streaming | Mensajes de IoT, eventos de IoT, propiedades de IoT | |
| [**Event Hub**](kusto/management/data-ingestion/eventhub.md) | [Formatos de datos compatibles](kusto/management/data-ingestion/eventhub.md#data-format) | N/D | Procesamiento por lotes, streaming | Mensajes, eventos | |
| [**Event Grid**](kusto/management/data-ingestion/eventgrid.md) | [Formatos de datos compatibles](kusto/management/data-ingestion/eventgrid.md#data-format) | 1  GB sin comprimir | Procesamiento por lotes | Ingesta continua desde Azure Storage, datos externos en Azure Storage | 100 KB es un tamaño de archivo óptimo, se usa tanto para cambiar el nombre de los blobs como para crearlos |
| [**Net Std**](net-standard-ingest-data.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes, streaming, directo | Escriba su propio código en función de las necesidades de la organización. |
| [**Python**](python-ingest-data.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes, streaming, directo | Escriba su propio código en función de las necesidades de la organización. |
| [**Node.js**](node-ingest-data.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes, streaming, directo | Escriba su propio código en función de las necesidades de la organización. |
| [**Java**](kusto/api/java/kusto-java-client-library.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes, streaming, directo | Escriba su propio código en función de las necesidades de la organización. |
| [**REST**](kusto/api/netfx/kusto-ingest-client-rest.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes, streaming, directo| Escriba su propio código en función de las necesidades de la organización. |
| [**Go**](kusto/api/golang/kusto-golang-client-library.md) | Se admiten todos los formatos | 1 GB sin comprimir (vea la nota) | Procesamiento por lotes, streaming, directo | Escriba su propio código en función de las necesidades de la organización. |

> [!Note] 
> Cuando se hace referencia a ella en la tabla anterior, la ingesta admite un tamaño de archivo máximo de 5 GB. Se recomienda ingerir archivos de entre 100 MB y 1 GB.

## <a name="ingestion-process"></a>Proceso de ingesta

Una vez que haya elegido el método de ingesta que más se ajuste a sus necesidades, siga estos pasos:

1. **Establecimiento de una directiva de retención**

    Los datos ingeridos en una tabla de Azure Data Explorer están sujetos a la directiva de retención vigente de la tabla. Salvo que la directiva de retención vigente se establezca explícitamente en una tabla, deriva de la directiva de retención de la base de datos. La retención activa es una función del tamaño del clúster y de la directiva de retención. Si el espacio disponible es insuficiente para la cantidad de datos que se ingieren se obligará a realizar una retención esporádica de los primeros datos.
    
    Asegúrese de que la directiva de retención de la base de datos se ajusta a sus necesidades. Si no es así, anúlela explícitamente en el nivel de tabla. Para más información, consulte [Directiva de retención](kusto/management/retentionpolicy.md). 
    
1. **de una tabla**

    Para poder ingerir datos, es preciso crear una tabla con antelación. Use una de las siguientes opciones:
   * Cree una tabla [con un comando](kusto/management/create-table-command.md). 
   * Cree una tabla con una [ingesta con un solo clic](one-click-ingestion-new-table.md).

    > [!Note]
    > Si un registro está incompleto o un campo no se puede analizar como tipo el de datos necesarios, las columnas de tabla correspondientes se rellenará con valores nulos.

1. **Creación de la asignación de esquemas**

    La [asignación de esquemas](kusto/management/mappings.md) ayuda a enlazar los campos de datos de origen a las columnas de la tabla de destino. La asignación permite tomar datos de distintos orígenes en la misma tabla, en función de los atributos definidos. Se admiten diferentes tipos de asignaciones, tanto orientadas a filas (CSV, JSON y AVRO) como orientadas a columnas (Parquet). En la mayoría de los métodos, las asignaciones también se pueden [crear previamente en la tabla](kusto/management/create-ingestion-mapping-command.md) y hacer referencia a ellas desde el parámetro de comando de ingesta.

1. **Establecimiento de una directiva de actualización** (opcional)

   Algunas de las asignaciones de formato de datos (Parquet, JSON y Avro) admiten transformaciones sencillas y útiles en el momento de la ingesta. Si el escenario requiere un procesamiento más complejo en el momento de la ingesta, use la directiva de actualización, lo que permite el procesamiento ligero mediante los comandos del lenguaje de consulta de Kusto. La directiva de actualización ejecuta automáticamente extracciones y transformaciones en los datos ingeridos en la tabla original e ingiere los datos resultantes en una o varias tablas de destino. Establezca la [directiva de actualización](kusto/management/update-policy.md).



## <a name="next-steps"></a>Pasos siguientes

* [Formatos de datos compatibles](ingestion-supported-formats.md)
* [Propiedades de la ingesta que se admiten](ingestion-properties.md)

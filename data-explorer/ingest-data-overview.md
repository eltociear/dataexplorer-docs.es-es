---
title: Ingesta de datos de Azure Data Explorer
description: Información acerca de las diferentes maneras que puede usar para la ingesta (carga) de datos en Azure Data Explorer
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/18/2019
ms.openlocfilehash: 8f48499e1b9bb1c67f43ae575a68bb38e314f08e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493120"
---
# <a name="azure-data-explorer-data-ingestion"></a>Ingesta de datos de Azure Data Explorer

La ingesta de datos es el proceso que se usa para cargar los registros de datos de uno o más orígenes para crear o actualizar una tabla en Azure Data Explorer. Una vez ingeridos, los datos están disponibles para su consulta. El diagrama siguiente muestra el flujo de un extremo a otro para trabajar en Azure Data Explorer, incluida la ingesta de datos.

![flujo de datos](media/ingest-data-overview/data-flow.png)

El servicio de administración de datos Azure Data Explorer, que es el responsable de la ingesta de datos, proporciona las siguientes funcionalidades:

1. **Extracción de datos**: extracción de datos de orígenes externos (Event Hubs) o lectura de las solicitudes de ingesta de una cola de Azure.

1. **Procesamiento por lotes**: procesamiento por lotes de datos que fluyen a la misma base de datos y tabla para optimizar el rendimiento de la ingesta de datos.

1. **Validación**: validación preliminar y conversión de formato, si es necesario.

1. **Manipulación de datos**: esquema coincidente, organización, indexación, codificación y compresión de los datos.

1. **Punto de persistencia en el flujo de ingesta**: administración de la carga de ingesta en el motor y control de los reintentos tras los errores transitorios.

1. **Confirmación de los datos de ingesta**: hace que los datos estén disponibles para su consulta.

## <a name="ingestion-methods"></a>Métodos de ingesta

Azure Data Explorer admite varios métodos de ingesta, cada uno con sus propios escenarios de destino, ventajas y desventajas. Azure Data Explorer ofrece canalizaciones y conectores para servicios comunes, ingesta mediante programación con los SDK y acceso directo al motor para fines de exploración.

### <a name="ingestion-using-pipelines-connectors-and-plugins"></a>Ingesta mediante canalizaciones, conectores y complementos

Actualmente, Azure Data Explorer es compatible con lo siguiente:

* Canalización de Event Grid, que se puede administrar con el asistente para administración en Azure Portal. Para más información, consulte [Ingesta de blobs de Azure en Azure Data Explorer](ingest-data-event-grid.md).

* Canalización del centro de eventos, que se puede administrar con el asistente para administración en Azure Portal. Para más información, consulte [Ingesta de datos desde el centro de eventos en Azure Data Explorer](ingest-data-event-hub.md).

* Complemento Logstash; consulte [Ingesta de datos de Logstash en Azure Data Explorer](ingest-data-logstash.md).

* Conector de Kafka; consulte [Ingesta de datos de Kafka en Azure Data Explorer](ingest-data-kafka.md).

### <a name="ingestion-using-integration-services"></a>Ingesta mediante servicios de integración

* Azure Data Factory (ADF), un servicio de integración de datos totalmente administrado para cargas de trabajo de análisis en Azure, para copiar datos con Azure Data Explorer como origen o destino usando [almacenes de datos y formatos compatibles](/azure/data-factory/copy-activity-overview#supported-data-stores-and-formats). Para obtener más información, consulte [Copiar datos de Azure Data Factory a Azure Data Explorer](/azure/data-explorer/data-factory-load-data).

### <a name="programmatic-ingestion"></a>Ingesta mediante programación

Azure Data Explorer proporciona SDK que pueden usarse para la consulta e ingesta de datos. La ingesta mediante programación está optimizada para reducir los costos de ingesta (COG), minimizando las transacciones de almacenamiento durante y después del proceso de ingesta.

**SDK y proyectos de código abierto disponibles**:

Kusto ofrece un cliente SDK que puede utilizarse para la ingesta y consulta de datos con:

* [SDK de Python](kusto/api/python/kusto-python-client-library.md)

* [SDK de .NET](kusto/api/netfx/about-the-sdk.md)

* [SDK de Java](kusto/api/java/kusto-java-client-library.md)

* [SDK de Node](kusto/api/node/kusto-node-client-library.md)

* [REST API](kusto/api/netfx/kusto-ingest-client-rest.md)

**Técnicas de ingesta mediante programación**:

* Ingesta de datos mediante el servicio de administración de datos Azure Data Explorer (ingesta confiable y de alto rendimiento):

    [**Ingesta de lotes**](kusto/api/netfx/kusto-ingest-queued-ingest-sample.md) (proporcionada por el SDK): el cliente carga los datos en Azure Blob Storage (designado por el servicio de administración de datos Azure Data Explorer) y envía una notificación a una cola de Azure. La ingesta de lotes es la técnica recomendada para la ingesta confiable y económica de datos de gran volumen.

* Ingesta de datos directamente en el motor de Azure Data Explorer (más adecuado para la exploración y la creación de prototipos):

  * **Ingesta insertada**: comando de control (.ingest inline) que contiene los datos en banda, está pensado para fines de pruebas ad hoc.

  * **Ingesta de consulta**: comando de control (.set, .set-or-append, .set-or-replace) que señala a los resultados de la consulta, se usa para generar informes o pequeñas tablas temporales.

  * **Ingesta de almacenamiento**: comando de control (.ingest into) con datos almacenados externamente (por ejemplo, Azure Blob Storage) permite la ingesta en masa de datos de forma eficaz.

**Latencia de los distintos métodos**:

| Método | Latencia |
| --- | --- |
| **Ingesta insertada** | Inmediata |
| **Ingesta de consulta** | Tiempo de consulta + tiempo de procesamiento |
| **Ingesta de almacenamiento** | Tiempo de descarga + tiempo de procesamiento |
| **Ingesta en cola** | Tiempo del procesamiento por lotes + tiempo de procesamiento |
| |

El tiempo de procesamiento depende del tamaño de los datos, de menos de unos segundos. El tiempo del procesamiento por lotes tiene un valor predeterminado de 5 minutos.

## <a name="choosing-the-most-appropriate-ingestion-method"></a>Selección del método más adecuado de ingesta

Antes de empezar con la ingesta de datos, debe plantearse las siguientes preguntas.

* ¿Dónde residen mis datos? 
* ¿Cuál es el formato de los datos y es posible cambiarlo? 
* ¿Cuáles son los campos obligatorios que se van a consultar? 
* ¿Cuál es el volumen de datos y la velocidad que se esperan? 
* ¿Cuántos tipos de eventos se esperan (reflejados como número de tablas)? 
* ¿Con qué frecuencia se espera que cambie el esquema de eventos? 
* ¿Cuántos nodos generarán los datos? 
* ¿Cuál es el SO del origen? 
* ¿Cuáles son los requisitos de latencia? 
* ¿Puede utilizarse una de las canalizaciones de ingesta administrada existentes? 

En el caso de las organizaciones con una infraestructura existente que usen un servicio de mensajería como Event Hub y IoT Hub, el uso de un conector es probablemente la solución más adecuada. La ingesta en cola es apropiada para grandes volúmenes de datos.

## <a name="supported-data-formats"></a>Formatos de datos compatibles

Para todos los métodos de ingesta, salvo la ingesta de consulta, cambie el formato de los datos para que Azure Data Explorer pueda analizarlos. 
* Los formatos de datos compatibles son: TXT, CSV, TSV, TSVE, PSV, SCSV, SOH, JSON (separados por línea, multilínea), Avro, Orc y Parquet. 
* Admite la compresión ZIP y GZIP.

> [!NOTE]
> Cuando se produce la ingesta de datos, los tipos de datos se infieren basándose en las columnas de tabla de destino. Si un registro está incompleto o un campo no se puede analizar como tipo el de datos necesarios, las columnas de tabla correspondientes se rellenará con valores nulos.

## <a name="ingestion-recommendations-and-limitations"></a>Limitaciones y recomendaciones de ingesta

* La directiva de retención efectiva de los datos ingeridos se deriva de la directiva de retención de la base de datos. Para información más detallada, consulte [Directiva de retención](kusto/management/retentionpolicy.md). La ingesta de datos requiere permisos de **Agente de ingesta de tablas** o de **Agente de ingesta de base de datos**.
* La ingesta admite un tamaño de archivo máximo de 5 GB. Se recomienda ingerir archivos de entre 100 MB y 1 GB.

## <a name="schema-mapping"></a>Asignación de esquemas

La asignación de esquemas, ayuda a enlazar campos de datos de origen a columnas de tabla de destino.

* [Asignación de CSV](kusto/management/mappings.md#csv-mapping) (opcional) funciona con todos los formatos basados en ordinales. se puede realizar mediante el parámetro de comando de ingesta o [se puede crear previamente en la tabla](kusto/management/create-ingestion-mapping-command.md) y hacerle referencia desde el parámetro de comando de ingesta.
* [Asignación de JSON](kusto/management/mappings.md#json-mapping) (obligatoria) y [Asignación de Avro](kusto/management/mappings.md#avro-mapping) (obligatoria) se pueden realizar mediante el parámetro de comando de ingesta. También se pueden [crear previamente en la tabla](kusto/management/create-ingestion-mapping-command.md) y referenciarse desde el parámetro de comando de ingesta.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ingesta de datos del centro de eventos a Azure Data Explorer](ingest-data-event-hub.md)

> [!div class="nextstepaction"]
> [Ingesta de datos mediante la suscripción de Event Grid en Azure Data Explorer](ingest-data-event-grid.md)

> [!div class="nextstepaction"]
> [Ingesta de datos de Kafka en Azure Data Explorer](ingest-data-kafka.md)

> [!div class="nextstepaction"]
> [Ingesta de datos mediante la biblioteca de Python de Azure Data Explorer](python-ingest-data.md)

> [!div class="nextstepaction"]
> [Ingesta de datos mediante la biblioteca de Node de Azure Data Explorer](node-ingest-data.md)

> [!div class="nextstepaction"]
> [Ingesta de datos mediante el SDK de .NET Standard de Azure Data Explorer (versión preliminar)](net-standard-ingest-data.md)

> [!div class="nextstepaction"]
> [Ingesta de datos de Logstash en Azure Data Explorer](ingest-data-logstash.md)

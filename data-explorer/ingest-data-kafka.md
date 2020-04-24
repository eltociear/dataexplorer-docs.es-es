---
title: Ingesta de datos de Kafka en Azure Data Explorer
description: En este artículo obtendrá información sobre cómo ingerir (cargar) datos en Azure Data Explorer desde Kafka.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/03/2019
ms.openlocfilehash: 03b46ff50683149a22c71ccb155480a0f08455bd
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493976"
---
# <a name="ingest-data-from-kafka-into-azure-data-explorer"></a>Ingesta de datos de Kafka en Azure Data Explorer
 
El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Azure Data Explorer ofrece ingesta (carga de datos) de Kafka. Kafka es una plataforma de streaming distribuida que permite la creación de canalizaciones de streaming de datos en tiempo real que mueven los datos de forma confiable entre aplicaciones o sistemas.
 
## <a name="prerequisites"></a>Prerequisites
 
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar. 
 
* [Una base de datos y un clúster de prueba](create-cluster-database-portal.md).
 
* [Una aplicación de ejemplo](https://github.com/Azure/azure-kusto-samples-dotnet/tree/master/kafka) que genera datos y los envía a Kafka.

* [Visual Studio de 2019](https://visualstudio.microsoft.com/vs/) para ejecutar la aplicación de ejemplo.
 
## <a name="kafka-connector-setup"></a>Instalación del conector de Kafka

Kafka Connect es una herramienta para realizar streaming de datos de forma escalable y confiable entre Apache Kafka y otros sistemas. Simplifica la definición rápida de conectores que mueven grandes colecciones de datos dentro y fuera de Kafka. El receptor de Kafka ADX actúa como conector de Kafka.
 
### <a name="bundle"></a>Agrupación

Kafka puede cargar un archivo `.jar` como complemento que funcionará como un conector personalizado. Para generar dicho archivo `.jar`, se clonará el código localmente y se compilará con Maven. 

#### <a name="clone"></a>Clonar

```bash
git clone git://github.com:Azure/kafka-sink-azure-kusto.git
cd ./kafka-sink-azure-kusto/kafka/
```

#### <a name="build"></a>Build

Compile localmente con Maven para generar un archivo `.jar` junto con dependencias.

* JDK >= 1.8 [descargar](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* Maven [descargar](https://maven.apache.org/install.html)
 

Dentro del directorio raíz *kafka-sink-azure-kusto*, ejecute:

```bash
mvn clean compile assembly:single
```

### <a name="deploy"></a>Implementación 

Cargue el complemento en Kafka. Se puede encontrar un ejemplo de implementación mediante Docker en [kafka-sink-azure-kusto](https://github.com/Azure/kafka-sink-azure-kusto#deploy)
 

Se puede encontrar documentación detallada sobre los conectores de Kafka y cómo implementarlos en [Kafka Connect](https://kafka.apache.org/documentation/#connect) 

### <a name="example-configuration"></a>Ejemplo de configuración 
 
```config
name=KustoSinkConnector 
connector.class=com.microsoft.azure.kusto.kafka.connect.sink.KustoSinkConnector 
kusto.sink.flush_interval_ms=300000 
key.converter=org.apache.kafka.connect.storage.StringConverter 
value.converter=org.apache.kafka.connect.storage.StringConverter 
tasks.max=1 
topics=testing1 
kusto.tables.topics_mapping=[{'topic': 'testing1','db': 'daniel', 'table': 'TestTable','format': 'json', 'mapping':'TestMapping'}] 
kusto.auth.authority=XXX 
kusto.url=https://ingest-{mycluster}.kusto.windows.net/ 
kusto.auth.appid=XXX 
kusto.auth.appkey=XXX 
kusto.sink.tempdir=/var/tmp/ 
kusto.sink.flush_size=1000
```
 
## <a name="create-a-target-table-in-adx"></a>Creación de una tabla de destino en ADX
 
Cree una tabla en ADX a la que Kafka pueda enviar datos. Cree la tabla en el clúster y la base de datos aprovisionada en **Requisitos previos**.
 
1. En Azure Portal, vaya al clúster y seleccione **Consultar**.
 
    ![Vínculo a la aplicación Consulta](media/ingest-data-event-hub/query-explorer-link.png)
 
1. Copie el siguiente comando en la ventana y seleccione **Ejecutar**.
 
    ```Kusto
    .create table TestTable (TimeStamp: datetime, Name: string, Metric: int, Source:string)
    ```
 
    ![Ejecución de creación de una consulta](media/ingest-data-event-hub/run-create-query.png)
 
1. Copie el siguiente comando en la ventana y seleccione **Ejecutar**.
 
    ```Kusto
    .create table TestTable ingestion json mapping 'TestMapping' '[{"column":"TimeStamp","path":"$.timeStamp","datatype":"datetime"},{"column":"Name","path":"$.name","datatype":"string"},{"column":"Metric","path":"$.metric","datatype":"int"},{"column":"Source","path":"$.source","datatype":"string"}]'
    ```

    Este comando asigna los datos JSON entrantes a los nombres de columna y tipos de datos de la tabla (TestTable).


## <a name="generate-sample-data"></a>Generación de datos de ejemplo

Ahora que el clúster de Kafka está conectado a ADX, use la [aplicación de ejemplo](https://github.com/Azure-Samples/event-hubs-dotnet-ingest) que descargó para generar datos.

### <a name="clone"></a>Clonar

Clone la aplicación de ejemplo localmente:

```cmd
git clone git://github.com:Azure/azure-kusto-samples-dotnet.git
cd ./azure-kusto-samples-dotnet/kafka/
```

### <a name="run-the-app"></a>Ejecución la aplicación

1. Abra la aplicación de ejemplo en Visual Studio.

1. En el archivo `Program.cs`, actualice la constante `connectionString` a la cadena de conexión de Kafka.

    ```csharp    
    const string connectionString = @"<YourConnectionString>";
    ```

1. Compile y ejecute la aplicación. La aplicación envía mensajes al clúster de Kafka e imprime su estado cada 10 segundos.

1. Después de la aplicación ha enviado algunos mensajes, continúe con el paso siguiente.
 
## <a name="query-and-review-the-data"></a>Consulta y revisión de los datos

1. Para asegurarse de que no se ha producido ningún error durante la ingesta:

    ```Kusto
    .show ingestion failures
    ```

1. Para ver los datos ingeridos recientemente:

    ```Kusto
    TestTable 
    | count
    ```

1. Para ver el contenido de los mensajes:
 
    ```Kusto
    TestTable
    ```
 
    El conjunto de resultados debe tener un aspecto similar al siguiente:
 
    ![Conjunto de resultados de mensajes](media/ingest-data-event-hub/message-result-set.png)
 
## <a name="next-steps"></a>Pasos siguientes
 
* [Consulta de datos en Azure Data Explorer](web-query-data.md)

---
title: Use el conector de Azure Data Explorer para Apache Spark para mover datos entre clústeres de Azure Data Explorer y Spark.
description: En este tema se muestra cómo mover datos entre clústeres de Azure Data Explorer y Apache Spark.
author: orspod
ms.author: orspodek
ms.reviewer: michazag
ms.service: data-explorer
ms.topic: conceptual
ms.date: 1/14/2020
ms.openlocfilehash: 6e7879948fb7683210859c84970f31bc5adb4a90
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108327"
---
# <a name="azure-data-explorer-connector-for-apache-spark"></a>Conector de Azure Data Explorer para Apache Spark

[Apache Spark](https://spark.apache.org/) es un motor de análisis unificado para el procesamiento de datos a gran escala. Azure Data Explorer es un servicio de análisis de datos rápido y totalmente administrado que permite analizar grandes volúmenes de datos en tiempo real. 

El conector de Azure Data Explorer para Spark es un [proyecto de código abierto](https://github.com/Azure/azure-kusto-spark) que se puede ejecutar en cualquier clúster de Spark. Implementa el origen y el receptor de datos para mover datos entre los clústeres de Azure Data Explorer y de Spark. Con Azure Data Explorer y Apache Spark, puede compilar aplicaciones rápidas y escalables orientadas a escenarios controlados por datos. Por ejemplo, aprendizaje automático (ML), extracción, transformación y carga de datos (ETL) y Log Analytics. Gracias al conector, Azure Data Explorer se convierte en un almacén de datos válido para las operaciones estándar de origen y receptor de Spark, como escritura, lectura y writeStream.

Puede escribir en Azure Data Explorer en modo de lote o de streaming. La lectura desde Azure Data Explorer admite la eliminación de columnas y la aplicación del predicado, lo que filtra los datos en Azure Data Explorer, reduciendo el volumen de datos transferidos.

En este tema se describe cómo instalar y configurar el conector de Azure Data Explorer para Spark y cómo trasladar los datos entre los clústeres de Azure Data Explorer y de Apache Spark.

> [!NOTE]
> Aunque algunos de los ejemplos siguientes se refieren a un clúster de Spark de [Azure Databricks](https://docs.azuredatabricks.net/), el conector de Spark de Azure Data Explorer no tiene dependencias directas en Databricks ni en ninguna otra distribución de Spark.

## <a name="prerequisites"></a>Prerrequisitos

* [Creación de un clúster y de la base de datos de Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal) 
* Creación de un clúster de Spark
* Instalación de la biblioteca de conectores de Azure Data Explorer:
    * Bibliotecas pregeneradas para [Spark 2.4, Scala 2.11](https://github.com/Azure/azure-kusto-spark/releases) 
    * [Repositorio de Maven](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/spark-kusto-connector)
* [Maven 3.x](https://maven.apache.org/download.cgi) instalado

> [!TIP]
> Las versiones 2.3.x también se admiten, pero pueden requerir algunos cambios en las dependencias pom.xml.

## <a name="how-to-build-the-spark-connector"></a>Cómo crear el conector de Spark

> [!NOTE]
> Este paso es opcional. Si utiliza bibliotecas pregeneradas, vaya a [Instalación de clúster de Spark](#spark-cluster-setup).

### <a name="build-prerequisites"></a>Requisitos previos de compilación

1. Instale las bibliotecas enumeradas en las [dependencias](https://github.com/Azure/azure-kusto-spark#dependencies) incluidas las siguientes de [SDK de Java de Kusto](kusto/api/java/kusto-java-client-library.md):
    * [Cliente de datos de Kusto](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-data)
    * [Cliente de ingesta de Kusto](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/kusto-ingest)

1. Consulte [este origen](https://github.com/Azure/azure-kusto-spark) para crear el conector de Spark.

1. Para las aplicaciones Scala y Java que utilizan definiciones de proyecto Maven, vincule la aplicación con el siguiente artefacto (la última versión puede ser diferente):
    
    ```Maven
       <dependency>
         <groupId>com.microsoft.azure</groupId>
         <artifactId>spark-kusto-connector</artifactId>
         <version>1.1.0</version>
       </dependency>
    ```

### <a name="build-commands"></a>Comandos de compilación

Para compilar el archivo jar y ejecutar todas las pruebas:

```
mvn clean package
```

Para compilar el archivo jar, ejecute todas las pruebas e instale jar en el repositorio de Maven local:

```
mvn clean install
```

Para obtener más información, consulte [Uso de conectores](https://github.com/Azure/azure-kusto-spark#usage).

## <a name="spark-cluster-setup"></a>Instalación del clúster de Spark

> [!NOTE]
> Se recomienda usar la última versión del conector de Azure Data Explorer para Spark al realizar los pasos a continuación.

1. Configure los siguientes valores del clúster de Spark, en función del clúster de Azure Databricks, mediante Spark 2.4.4 y Scala 2.11:

    ![Configuración del clúster de Databricks](media/spark-connector/databricks-cluster.png)
    
1. Instale la biblioteca spark-kusto-connector más reciente de Maven:
    
    ![Importación de bibliotecas](media/spark-connector/db-libraries-view.png) ![Selección de Spark-Kusto-Connector](media/spark-connector/db-dependencies.png)

1. Compruebe que están instaladas todas las bibliotecas necesarias:

    ![Comprobación de las bibliotecas instaladas](media/spark-connector/db-libraries-view.png)

1. Para realizar la instalación con un archivo JAR, compruebe que se instalaron dependencias adicionales:

    ![Adición de dependencias](media/spark-connector/db-not-maven.png)

## <a name="authentication"></a>Authentication

El conector de Azure Data Explorer para Spark permite autenticarse con Azure Active Directory (Azure AD) mediante uno de los métodos siguientes:
* Una [aplicación de Azure AD](#azure-ad-application-authentication)
* Un [token de acceso de Azure AD](https://github.com/Azure/azure-kusto-spark/blob/master/docs/Authentication.md#direct-authentication-with-access-token)
* [Autenticación de dispositivos](https://github.com/Azure/azure-kusto-spark/blob/master/docs/Authentication.md#device-authentication) (para escenarios que no sean de producción)
* Una instancia de [Azure Key Vault](https://github.com/Azure/azure-kusto-spark/blob/master/docs/Authentication.md#key-vault). Para acceder al recurso de Key Vault, instale el paquete azure-keyvault y proporcione las credenciales de aplicación.

### <a name="azure-ad-application-authentication"></a>Autenticación de la aplicación de Azure AD

La autenticación de la aplicación de Azure AD es el método de autenticación más sencillo y más común, y se recomienda para el conector de Azure Data Explorer para Spark.

|Propiedades  |Descripción  |
|---------|---------|
|**KUSTO_AAD_CLIENT_ID**     |   Identificador de la aplicación de Azure AD (cliente).      |
|**KUSTO_AAD_AUTHORITY_ID**     |  Autoridad de autenticación de Azure AD. Identificador de Azure AD Directory (inquilino).        |
|**KUSTO_AAD_CLIENT_PASSWORD**    |    Clave de aplicación de Azure AD para el cliente.     |

### <a name="azure-data-explorer-privileges"></a>Privilegios de Azure Data Explorer

Conceda los privilegios a continuación a un clúster de Azure Data Explorer:

* Para lectura (origen de datos), la aplicación de Azure AD tiene que tener privilegios de *espectador* en la base de datos de destino, o de *administrador* en la tabla de destino.
* Para escritura (receptor de datos), la aplicación de Azure AD tiene que tener privilegios de *agente de ingesta* en la base de datos de destino. También debe tener privilegios de *usuario* en la base de datos de destino para crear nuevas tablas. Si la tabla de destino ya existe, se tienen que configurar privilegios de *administrador* en la tabla de destino.
 
Para obtener más información sobre los roles principales de Azure Data Explorer, consulte [Autorización basada en roles](kusto/management/access-control/role-based-authorization.md). Para administrar los roles de seguridad, consulte [Administración de los roles de seguridad](kusto/management/security-roles.md).

## <a name="spark-sink-writing-to-azure-data-explorer"></a>Receptor de Spark: escritura en Azure Data Explorer

1. Configuración de los parámetros del receptor:

     ```scala
    val KustoSparkTestAppId = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppId")
    val KustoSparkTestAppKey = dbutils.secrets.get(scope = "KustoDemos", key = "KustoSparkTestAppKey")
 
    val appId = KustoSparkTestAppId
    val appKey = KustoSparkTestAppKey
    val authorityId = "72f988bf-86f1-41af-91ab-2d7cd011db47" // Optional - defaults to microsoft.com
    val cluster = "Sparktest.eastus2"
    val database = "TestDb"
    val table = "StringAndIntTable"
    ```

1. Escriba DataFrame de Spark en clústeres de Azure Data Explorer como lote:

    ```scala
    import com.microsoft.kusto.spark.datasink.KustoSinkOptions
    import org.apache.spark.sql.{SaveMode, SparkSession}

    df.write
      .format("com.microsoft.kusto.spark.datasource")
      .option(KustoSinkOptions.KUSTO_CLUSTER, cluster)
      .option(KustoSinkOptions.KUSTO_DATABASE, database)
      .option(KustoSinkOptions.KUSTO_TABLE, "Demo3_spark")
      .option(KustoSinkOptions.KUSTO_AAD_CLIENT_ID, appId)
      .option(KustoSinkOptions.KUSTO_AAD_CLIENT_PASSWORD, appKey)
      .option(KustoSinkOptions.KUSTO_AAD_AUTHORITY_ID, authorityId)
      .option(KustoSinkOptions.KUSTO_TABLE_CREATE_OPTIONS, "CreateIfNotExist")
      .mode(SaveMode.Append)
      .save()  
    ```
    
   O bien, use la sintaxis simplificada:
   
    ```scala
         import com.microsoft.kusto.spark.datasink.SparkIngestionProperties
         import com.microsoft.kusto.spark.sql.extension.SparkExtension._
         
         val sparkIngestionProperties = Some(new SparkIngestionProperties()) // Optional, use None if not needed
         df.write.kusto(cluster, database, table, conf, sparkIngestionProperties)
    ```
   
1. Escriba datos de streaming:

    ```scala    
    import org.apache.spark.sql.streaming.Trigger
    import java.util.concurrent.TimeUnit
    import java.util.concurrent.TimeUnit
    import org.apache.spark.sql.streaming.Trigger

    // Set up a checkpoint and disable codeGen. 
    spark.conf.set("spark.sql.streaming.checkpointLocation", "/FileStore/temp/checkpoint")
        
    // Write to a Kusto table from a streaming source
    val kustoQ = df
          .writeStream
          .format("com.microsoft.kusto.spark.datasink.KustoSinkProvider")
          .options(conf) 
          .option(KustoSinkOptions.KUSTO_WRITE_ENABLE_ASYNC, "true") // Optional, better for streaming, harder to handle errors
          .trigger(Trigger.ProcessingTime(TimeUnit.SECONDS.toMillis(10))) // Sync this with the ingestionBatching policy of the database
          .start()
    ```

## <a name="spark-source-reading-from-azure-data-explorer"></a>Origen de Spark: lectura desde Azure Data Explorer

1. Al leer [pequeñas cantidades de datos](kusto/concepts/querylimits.md), defina la consulta de datos:

    ```scala
    import com.microsoft.kusto.spark.datasource.KustoSourceOptions
    import org.apache.spark.SparkConf
    import org.apache.spark.sql._
    import com.microsoft.azure.kusto.data.ClientRequestProperties

    val query = s"$table | where (ColB % 1000 == 0) | distinct ColA"
    val conf: Map[String, String] = Map(
          KustoSourceOptions.KUSTO_AAD_CLIENT_ID -> appId,
          KustoSourceOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey
        )

    val df = spark.read.format("com.microsoft.kusto.spark.datasource").
      options(conf).
      option(KustoSourceOptions.KUSTO_QUERY, query).
      option(KustoSourceOptions.KUSTO_DATABASE, database).
      option(KustoSourceOptions.KUSTO_CLUSTER, cluster).
      load()

    // Simplified syntax flavor
    import com.microsoft.kusto.spark.sql.extension.SparkExtension._
    
    val cpr: Option[ClientRequestProperties] = None // Optional
    val df2 = spark.read.kusto(cluster, database, query, conf, cpr)
    display(df2)
    ```

1. Opcional: Si es **usted** (y no Azure Data Explorer) el que proporciona el almacenamiento de blobs transitorio, la responsabilidad de la creación de los blobs es del autor de llamada. Esto incluye el aprovisionamiento del almacenamiento, la rotación de claves de acceso y la eliminación de artefactos transitorios. 
    El módulo KustoBlobStorageUtils contiene funciones de asistente para eliminar blobs en función de las coordenadas de cuenta y de contenedor y las credenciales de cuenta, o de una dirección URL de SAS completa con permisos de escritura, lectura y lista. Cuando ya no se necesita el RDD correspondiente, cada transacción almacena artefactos de blob transitorios en un directorio independiente. Este directorio se captura como parte de los registros de información de transacciones de lectura que se notifica en el nodo del controlador de Spark.

    ```scala
    // Use either container/account-key/account name, or container SaS
    val container = dbutils.secrets.get(scope = "KustoDemos", key = "blobContainer")
    val storageAccountKey = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountKey")
    val storageAccountName = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageAccountName")
    // val storageSas = dbutils.secrets.get(scope = "KustoDemos", key = "blobStorageSasUrl")
    ```

    En el ejemplo anterior, no se accede a Key Vault mediante la interfaz del conector; se usa un método más sencillo con los secretos de Databricks.

1. Lectura desde Azure Data Explorer.

    * Si **usted** proporciona el almacenamiento de blobs transitorio, lea desde Azure Data Explorer de la siguiente manera:

        ```scala
         val conf3 = Map(
              KustoSourceOptions.KUSTO_AAD_CLIENT_ID -> appId,
              KustoSourceOptions.KUSTO_AAD_CLIENT_PASSWORD -> appKey
              KustoSourceOptions.KUSTO_BLOB_STORAGE_SAS_URL -> storageSas)
        val df2 = spark.read.kusto(cluster, database, "ReallyBigTable", conf3)
        
        val dfFiltered = df2
          .where(df2.col("ColA").startsWith("row-2"))
          .filter("ColB > 12")
          .filter("ColB <= 21")
          .select("ColA")
        
        display(dfFiltered)
        ```

    * Si **Azure Data Explorer** proporciona el almacenamiento de blobs transitorio, lea desde Azure Data Explorer como se indica a continuación:
    
        ```scala
        val dfFiltered = df2
          .where(df2.col("ColA").startsWith("row-2"))
          .filter("ColB > 12")
          .filter("ColB <= 21")
          .select("ColA")
        
        display(dfFiltered)
        ```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [conector de Azure Data Explorer para Spark](https://github.com/Azure/azure-kusto-spark/tree/master/docs)
* [Código de ejemplo para Java y Python](https://github.com/Azure/azure-kusto-spark/tree/master/samples/src/main)

---
title: LightIngest es una utilidad de línea de comandos para la ingesta en Azure Data Explorer.
description: Más información acerca de LightIngest, una utilidad de línea de comandos para la ingesta de datos ad-hoc en Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/01/2020
ms.openlocfilehash: 8d4eeb47abb8eac2b042b64e65b55dac7e91d6c9
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83374052"
---
# <a name="install-and-use-lightingest"></a>Instalación y uso de LightIngest

LightIngest es una utilidad de línea de comandos para la ingesta de datos ad-hoc en Azure Data Explorer.
La utilidad puede extraer datos de origen de una carpeta local o de un contenedor de almacenamiento de blobs de Azure.

## <a name="prerequisites"></a>Prerrequisitos

* LightIngest: descárguelo como parte del [paquete NuGet Microsoft.Azure.Kusto.Tools](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/)

    ![Descarga de LightIngest](media/lightingest/lightingest-download-area.png)

* WinRAR: descárguelo de [www.win-rar.com/download.html](http://www.win-rar.com/download.html)

## <a name="install-lightingest"></a>Instalación de LightIngest

1. Vaya a la ubicación del equipo en donde descargó LightIngest. 
1. Con WinRAR, extraiga el directorio *tools* en el equipo.

## <a name="run-lightingest"></a>Ejecución de LightIngest

1. Vaya al directorio *tools* que ha extraído en el equipo.
1. En la barra de ubicación elimine la información de ubicación existente.
    
      ![Eliminación de la información de ubicación](media/lightingest/lightingest-location-bar.png)

1. Escriba `cmd` y presione **ENTRAR**.
1. En el símbolo del sistema, escriba `LightIngest.exe` seguido del argumento de línea de comandos apropiado.

    > [!Tip]
    > Para obtener una lista de los argumentos de línea de comandos admitidos, escriba `LightIngest.exe /help`.
    >
    >![Ayuda de la línea de comandos](media/lightingest/lightingest-cmd-line-help.png)

1. Escriba `ingest-` seguido de la cadena de conexión al clúster de Azure Data Explorer que administrará la ingesta.
    Ponga la cadena de conexión entre comillas dobles y siga la [especificación de las cadenas de conexión de Kusto](kusto/api/connection-strings/kusto.md).

    Por ejemplo:
    ```
    ingest-{Cluster name and region}.kusto.windows.net;AAD Federated Security=True -db:{Database} -table:Trips -source:"https://{Account}.blob.core.windows.net/{ROOT_CONTAINER};{StorageAccountKey}" -pattern:"*.csv.gz" -format:csv -limit:2 -ignoreFirst:true -cr:10.0 -dontWait:true
    ```

* El método recomendado permite que LightIngest funcione con el punto de conexión de ingesta en `https://ingest-{yourClusterNameAndRegion}.kusto.windows.net`. De esta manera, el servicio Azure Data Explorer puede administrar la carga de ingesta, y usted puede recuperarse fácilmente de los errores transitorios. De todas formas, también puede configurar LightIngest para que trabaje directamente con el punto de conexión del motor (`https://{yourClusterNameAndRegion}.kusto.windows.net`).

> [!Note]
> Si la ingesta se realiza directamente con el punto de conexión del motor, no es necesario incluir `ingest-`, aunque no habrá ninguna característica de DM para proteger el motor y mejorar la tasa de éxito de la ingesta.

* Para obtener un rendimiento óptimo en la ingesta, es importante que LightIngest conozca el tamaño de los datos sin procesar, para que pueda calcular el tamaño sin comprimir de los archivos locales. De todos modos es posible que LightIngest no pueda calcular correctamente el tamaño sin procesar de los blobs comprimidos sin descargarlos primero. Por lo tanto, cuando realice la ingesta de blobs comprimidos, establezca la propiedad `rawSizeBytes` de los metadatos de blob en el tamaño de los datos sin comprimir en bytes.

## <a name="general-command-line-arguments"></a>Argumentos generales de la línea de comandos

|Nombre de argumento         |Nombre corto   |Tipo    |Mandatory |Descripción                                |
|----------------------|-------------|--------|----------|-------------------------------------------|
|                      |             |string  |Mandatory |[Cadena de conexión de Azure Data Explorer](kusto/api/connection-strings/kusto.md) que especifica el punto de conexión de Kusto que controlará la ingesta. Debe ir entre comillas dobles |
|-database             |-db          |string  |Opcional  |Nombre de la base de datos de Azure Data Explorer de destino |
|-table                |             |string  |Mandatory |Nombre de tabla de Azure Data Explorer de destino |
|-sourcePath           |-source      |string  |Mandatory |Ruta de acceso a los archivos de origen o URI raíz del contenedor de blobs. Si los datos están en blobs, tiene que contener una clave de cuenta de almacenamiento o una SAS. Se recomienda poner entre comillas dobles |
|-prefix               |             |string  |Opcional  |Cuando los datos de origen para la ingesta residen en un almacenamiento de blobs, todos los blobs comparten este prefijo de dirección URL, excepto el nombre del contenedor. <br>Por ejemplo, si los datos están en `MyContainer/Dir1/Dir2`, el prefijo debe ser `Dir1/Dir2`. Se recomienda poner entre comillas dobles |
|-pattern              |             |string  |Opcional  |Patrón que se sigue para seleccionar los archivos o blobs de origen. Admite caracteres comodín. Por ejemplo, `"*.csv"`. Se recomienda poner entre comillas dobles |
|-zipPattern           |             |string  |Opcional  |Expresión regular que se usa al seleccionar cuáles son los archivos de un archivo ZIP que se van a ingerir.<br>Los restantes archivos del archivo se omitirán. Por ejemplo, `"*.csv"`. Se recomienda poner entre comillas dobles |
|-format               |-f           |string  |Opcional  |Formato de datos de origen. Tiene que ser uno de los [formatos admitidos](ingestion-supported-formats.md) |
|-ingestionMappingPath |-mappingPath |string  |Opcional  |Ruta de acceso al archivo de asignación de columnas de ingesta (obligatorio para los formatos JSON y Avro). Consulte [Asignaciones de datos](kusto/management/mappings.md) |
|-ingestionMappingRef  |-mappingRef  |string  |Opcional  |Nombre de una asignación de columna de ingesta creada previamente (obligatorio para los formatos JSON y Avro). Consulte [Asignaciones de datos](kusto/management/mappings.md) |
|-creationTimePattern  |             |string  |Opcional  |Cuando se establece, se usa para extraer la propiedad CreationTime de la ruta de acceso del archivo o del blob. Consulte [Uso del argumento CreationTimePattern](#using-creationtimepattern-argument) |
|-ignoreFirstRow       |-ignoreFirst |bool    |Opcional  |Si está establecido, se omite el primer registro de cada archivo o blob (por ejemplo, si los datos de origen tienen encabezados) |
|-tag                  |             |string  |Opcional  |[Etiquetas](kusto/management/extents-overview.md#extent-tagging) para asociar con los datos ingeridos. Se permiten repeticiones |
|-dontWait             |             |bool    |Opcional  |Si se establece en "true", no espera a la finalización de la ingesta. Resulta útil cuando se realiza la ingesta de grandes cantidades de archivos o blobs |

### <a name="using-creationtimepattern-argument"></a>Uso del argumento CreationTimePattern

El argumento `-creationTimePattern` extrae la propiedad CreationTime de la ruta de acceso del archivo o del blob. No es necesario que el patrón refleje la ruta de acceso completa del elemento, basta con la sección que contiene la marca de tiempo que se quiere usar.

Entre los valores de argumento se debe incluir lo siguiente:
* Prueba de constantes inmediatamente antes de la marca de tiempo, encerrada entre comillas simples
* El formato de marca de tiempo, en [notación DateTime de .NET](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings) estándar.
* Texto de constante que sigue inmediatamente a la marca de tiempo. Por ejemplo, si los nombres de blob terminan con `historicalvalues19840101.parquet` (la marca de tiempo tiene cuatro dígitos para el año, dos dígitos para el mes y dos dígitos para el día del mes), el valor correspondiente para el argumento `-creationTimePattern` es:

```
ingest-{Cluster name and region}.kusto.windows.net;AAD Federated Security=True -db:{Database} -table:Trips -source:"https://{Account}.blob.core.windows.net/{ROOT_CONTAINER};{StorageAccountKey}" -creationTimePattern:"'historicalvalues'yyyyMMdd'.parquet'"
 -pattern:"*.csv.gz" -format:csv -limit:2 -ignoreFirst:true -cr:10.0 -dontWait:true
```

### <a name="command-line-arguments-for-advanced-scenarios"></a>Argumentos de la línea de comandos para escenarios avanzados

|Nombre de argumento         |Nombre corto   |Tipo    |Mandatory |Descripción                                |
|----------------------|-------------|--------|----------|-------------------------------------------|
|-compression          |-cr          |double  |Opcional  |Sugerencia de razón de compresión. Resulta útil cuando se realiza la ingesta de archivos o blobs comprimidos para ayudar a Azure Data Explorer a evaluar el tamaño de los datos sin procesar. Calculada como el tamaño original dividido por el tamaño comprimido |
|-limit                |-l           |integer |Opcional  |Si se establece, limita la ingesta a los primeros N archivos |
|-listOnly             |-list        |bool    |Opcional  |Si se establece, solo muestra los elementos que se hubieran seleccionado para la ingesta| 
|-ingestTimeout        |             |integer |Opcional  |Tiempo de expiración en minutos para la finalización de todas las operaciones de ingesta. De manera predeterminada, su valor es `60`.|
|-forceSync            |             |bool    |Opcional  |Si se establece, fuerza la ingesta sincrónica. De manera predeterminada, su valor es `false`. |
|-dataBatchSize        |             |integer |Opcional  |Establece el límite de tamaño total (MB, sin comprimir) de cada operación de ingesta |
|-filesInBatch         |             |integer |Opcional  |Establece el límite de recuento de archivos o blobs de cada operación de ingesta |
|-devTracing           |-trace       |string  |Opcional  |Si se establece, los registros de diagnóstico se escriben en un directorio local (de forma predeterminada, `RollingLogs` en el directorio actual, o se pueden modificar estableciendo el valor del modificador) |

## <a name="blob-metadata-properties"></a>Propiedades de metadatos de blob
Cuando se usa con blobs de Azure, LightIngest usa ciertas propiedades de metadatos de blob para aumentar el proceso de ingesta.

|Propiedad de metadatos                            | Uso                                                                           |
|---------------------------------------------|---------------------------------------------------------------------------------|
|`rawSizeBytes`, `kustoUncompressedSizeBytes` | Si se establece, se interpretará como el tamaño de los datos sin comprimir                       |
|`kustoCreationTime`, `kustoCreationTimeUtc`  | Se interpreta como una marca de tiempo en formato UTC. Si se establece, se usará para invalidar la hora de creación en Kusto. Útil para escenarios de reposición |

## <a name="usage-examples"></a>Ejemplos de uso

<!-- Waiting for Tzvia or Vladik to rewrite the instructions for this example before publishing it

### Ingesting a specific number of blobs in JSON format

* Ingest two blobs under a specified storage account {Account}, in `JSON` format matching the pattern `.json`
* Destination is the database {Database}, the table `SampleData`
* Indicate that your data is compressed with the approximate ratio of 10.0
* LightIngest won't wait for the ingestion to be completed

To use the LightIngest command below:
1. Create a table command and enter the table name into the LightIngest command, replacing `SampleData`.
1. Create a mapping command and enter the IngestionMappingRef command, replacing `SampleData_mapping`.
1. Copy your cluster name and enter it into the LightIngest command, replacing `{ClusterandRegion}`.
1. Enter the database name into the LightIngest command, replacing `{Database name}`.
1. Replace `{Account}` with your account name and replace `{ROOT_CONTAINER}?{SAS token}` with the appropriate information.

    ```
    LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.windows.net;Fed=True"  
        -db:{Database name} 
        -table:SampleData 
        -source:"https://{Account}.blob.core.windows.net/{ROOT_CONTAINER}?{SAS token}" 
        -IngestionMappingRef:SampleData_mapping 
        -pattern:"*.json" 
        -format:JSON 
        -limit:2 
        -cr:10.0 
        -dontWait:true
    ```
     
1. In Azure Data Explorer, open query count.

    ![Injestion result in Azure Data Explorer](media/lightingest/lightingest-show-failure-count.png)
-->

### <a name="ingesting-blobs-using-a-storage-account-key-or-a-sas-token"></a>Ingesta de blobs con una clave de cuenta de almacenamiento o un token de SAS

* Realice la ingesta de 10 blobs en la cuenta de almacenamiento especificada `ACCOUNT`, en la carpeta `DIR`, dentro del contenedor `CONT`, y que coincidan con el patrón `*.csv.gz`
* El destino es una base de datos `DB`, tabla `TABLE` y la asignación de ingesta `MAPPING` ya se ha creado con anterioridad en el destino
* La herramienta esperará hasta que se completen las operaciones de ingesta
* Tenga en cuenta las diferentes opciones para especificar la base de datos de destino y la clave de la cuenta de almacenamiento frente al Token de SAS

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.windows.net;Fed=True"
  -database:DB
  -table:TABLE
  -source:"https://ACCOUNT.blob.core.windows.net/{ROOT_CONTAINER};{StorageAccountKey}"
  -prefix:"DIR"
  -pattern:*.csv.gz
  -format:csv
  -mappingRef:MAPPING
  -limit:10

LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.windows.net;Fed=True;Initial Catalog=DB"
  -table:TABLE
  -source:"https://ACCOUNT.blob.core.windows.net/{ROOT_CONTAINER}?{SAS token}"
  -prefix:"DIR"
  -pattern:*.csv.gz
  -format:csv
  -mappingRef:MAPPING
  -limit:10
```

### <a name="ingesting-all-blobs-in-a-container-not-including-header-rows"></a>Ingesta de todos los blobs de un contenedor, sin incluir las filas de encabezado

* Realice la ingesta de todos los blobs de la cuenta de almacenamiento especificada `ACCOUNT`, en la carpeta `DIR1/DIR2`, dentro del contenedor `CONT`, y que coincidan con el patrón `*.csv.gz`
* El destino es la base de datos `DB`, tabla `TABLE` y la asignación de ingesta `MAPPING` ya se ha creado con anterioridad en el destino
* Los blobs de origen contienen una línea de encabezado, por lo que se indica a la herramienta que anule el primer registro de cada blob
* La herramienta publicará los datos de la ingesta y no esperará a que se completen las operaciones de esta

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.windows.net;Fed=True"
  -database:DB
  -table:TABLE
  -source:"https://ACCOUNT.blob.core.windows.net/{ROOT_CONTAINER}?{SAS token}"
  -prefix:"DIR1/DIR2"
  -pattern:*.csv.gz
  -format:csv
  -mappingRef:MAPPING
  -ignoreFirstRow:true
```

### <a name="ingesting-all-json-files-from-a-path"></a>Ingesta de todos los archivos JSON de una ruta de acceso

* Realice la ingesta de todos los archivos de la ruta de acceso `PATH`, que coinciden con el patrón `*.json`
* El destino es la base de datos `DB`, tabla `TABLE` y la asignación de ingesta está definida en el archivo local `MAPPING_FILE_PATH`
* La herramienta publicará los datos de la ingesta y no esperará a que se completen las operaciones de esta

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.windows.net;Fed=True"
  -database:DB
  -table:TABLE
  -source:"PATH"
  -pattern:*.json
  -format:json
  -mappingPath:"MAPPING_FILE_PATH"
```

### <a name="ingesting-files-and-writing-diagnostic-trace-files"></a>Ingesta de archivos y escritura de archivos de seguimiento de diagnóstico

* Realice la ingesta de todos los archivos de la ruta de acceso `PATH`, que coinciden con el patrón `*.json`
* El destino es la base de datos `DB`, tabla `TABLE` y la asignación de ingesta está definida en el archivo local `MAPPING_FILE_PATH`
* La herramienta publicará los datos de la ingesta y no esperará a que se completen las operaciones de esta
* Los archivos de seguimiento de diagnóstico se escribirán localmente en la carpeta `LOGS_PATH`

```
LightIngest.exe "https://ingest-{ClusterAndRegion}.kusto.windows.net;Fed=True"
  -database:DB
  -table:TABLE
  -source:"PATH"
  -pattern:*.json
  -format:json
  -mappingPath:"MAPPING_FILE_PATH"
  -trace:"LOGS_PATH"
```
## <a name="changelog"></a>Registro de cambios
|Versión        |Cambios                                                                             |
|---------------|------------------------------------------------------------------------------------|
|4.0.9.0        |<ul><li>Se ha agregado el argumento `-zipPattern`</li><li>Se ha agregado el argumento `-listOnly`</li><li>El resumen de los argumentos se muestra antes de que se inicie la ejecución</li><li>CreationTime se lee desde las propiedades de metadatos de blob o desde el nombre de archivo o de blob, según el argumento `-creationTimePattern`</li></ul>|

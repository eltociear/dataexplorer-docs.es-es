---
title: Formatos de datos que admite Azure Data Explorer para la ingesta.
description: Obtenga información sobre los diversos formatos de datos y compresión compatibles con Azure Data Explorer para la ingesta.
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/19/2020
ms.openlocfilehash: 93451dd1237905754ac834cb335f5d8a02157ac7
ms.sourcegitcommit: 29018b3db4ea7d015b1afa65d49ecf918cdff3d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "82030062"
---
# <a name="data-formats-supported-by-azure-data-explorer-for-ingestion"></a>Formatos de datos que admite Azure Data Explorer para la ingesta

La ingesta de datos es el proceso por el que se agregan datos a una tabla y se ponen a disposición para su consulta en Azure Data Explorer. En el caso de los métodos de ingesta que no sean la ingesta desde consulta, los datos deben tener uno de los formatos admitidos. En la siguiente tabla se enumeran y describen los formatos que admite Azure Data Explorer para la ingesta de datos.

|Formato   |Extensión   |Descripción|
|---------|------------|-----------|
|Avro     |`.avro`     |Un [archivo del contenedor Avro](https://avro.apache.org/docs/current/). Se admiten los siguientes códigos: `null`, `deflate` (`snappy` no se admite actualmente).|
|ApacheAvro|`.avro`    |Implementación nativa experimental para el formato [Avro](https://avro.apache.org/docs/current/) compatible con [tipos lógicos](https://avro.apache.org/docs/current/spec.html#Logical+Types) y con el códec de compresión `snappy`.|
|CSV      |`.csv`      |Un archivo de texto con valores separados por comas (`,`). Consulte [RFC 4180: _Formato común y tipo MIME para archivos de valores separados por comas (CSV)_ ](https://www.ietf.org/rfc/rfc4180.txt).|
|JSON     |`.json`     |Un archivo de texto con objetos JSON delimitados por `\n` o `\r\n`. Consulte [JSON Lines (JSONL)](http://jsonlines.org/).|
|MultiJSON|`.multijson`|Un archivo de texto con una matriz JSON de contenedores de propiedades (cada uno de los cuales representa un registro) o cualquier número de contenedores de propiedades delimitados por espacios en blanco, `\n` o `\r\n`. Cada contenedor de propiedades se puede distribuir en varias líneas. Este formato se prefiere a `JSON`, salvo que los datos sean contenedores sin propiedades.|
|ORC      |`.orc`      |Un [archivo Orc](https://en.wikipedia.org/wiki/Apache_ORC).|
|Parquet  |`.parquet`  |Un [archivo Parquet](https://en.wikipedia.org/wiki/Apache_Parquet).|
|PSV      |`.psv`      |Un archivo de texto con valores separados por barras verticales (<code>&#124;</code>).|
|RAW      |`.raw`      |Un archivo de texto cuyo contenido completo es un valor de cadena único.|
|SCsv     |`.scsv`     |Un archivo de texto con valores separados por punto y coma (`;`).|
|SOHsv    |`.sohsv`    |Un archivo de texto con valores separados por SOH. (SOH es un punto de código ASCII 1; este formato lo usa Hive en HDInsight).|
|TSV      |`.tsv`      |Un archivo de texto con valores separados por tabulaciones (`\t`).|
|TSVE     |`.tsv`      |Un archivo de texto con valores separados por tabulaciones (`\t`). Se utiliza un carácter de barra diagonal inversa (`\`) para el escape.|
|TXT      |`.txt`      |Un archivo de texto con líneas delimitadas por `\n`. Se omiten las líneas vacías.|

## <a name="supported-data-compression-formats"></a>Formatos de compresión de datos admitidos

Tanto los blobs como los archivos se pueden comprimir con cualquiera de los siguientes algoritmos de compresión:

|Compresión|Extensión|
|-----------|---------|
|GZip       |.gz      |
|Zip        |.zip     |

Indique la compresión anexando la extensión al nombre del blob o archivo.

Por ejemplo:
* `MyData.csv.zip` indica un blob o archivo con formato CSV, comprimido con ZIP (varios archivos o uno archivo).
* `MyData.csv.gz` indica un blob o un archivo con formato CSV, comprimido con GZip.

Los nombres de blobs o archivos que no incluyen extensiones de formato y solo incluyen la compresión (por ejemplo, `MyData.zip`) también se admiten. En este caso, el formato de archivo se debe especificar como una propiedad de la ingesta ya que no se puede deducir.

> [!NOTE]
> Algunos formatos de compresión mantienen un seguimiento de la extensión de archivo original como parte del flujo de compresión. Normalmente, esta extensión se omite para determinar el formato de archivo. Si este no se puede determinar a partir del nombre de archivo o blob (comprimido), debe especificarse mediante la propiedad de ingesta `format`.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre la [ingesta de datos](/azure/data-explorer/ingest-data-overview)
* Más información sobre las [Propiedades de la ingesta de datos de Azure Data Explorer](ingestion-properties.md)

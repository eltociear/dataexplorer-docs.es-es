---
title: 'Extensiones (particiones de datos): Azure Explorador de datos'
description: En este artículo se describen las extensiones (particiones de datos) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/13/2020
ms.openlocfilehash: 6c7910a222227dbb6b22e1fc4f0f4136897a0ef9
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84665119"
---
# <a name="extents-data-shards"></a>Extensiones (particiones de datos)

## <a name="overview"></a>Información general

Kusto se ha creado para admitir tablas con un gran número de registros (filas) y grandes cantidades de datos. Para administrar tablas de gran tamaño, los datos de cada tabla se dividen en "tabletas" más pequeñas denominadas **particiones de datos** o **extensiones** (los dos términos son sinónimos). La Unión de todas las extensiones de la tabla contiene los datos de la tabla. Las extensiones individuales se mantienen más pequeñas que la capacidad de un solo nodo y las extensiones se reparten a través de los nodos del clúster, con lo que se consigue el escalado horizontal.

Una extensión es similar a un tipo de tabla. Contiene datos y metadatos e información, como la hora de creación y las etiquetas opcionales asociadas a sus datos. Además, la extensión normalmente contiene información que permite a Kusto consultar los datos de forma eficaz.
Por ejemplo, un índice para cada columna de datos en la extensión y un diccionario de codificación, si se codifican los datos de la columna. Como resultado, los datos de la tabla son la Unión de todos los datos de las extensiones de la tabla.

Las extensiones son inmutables y nunca se pueden modificar. Solo puede consultarse, reasignarse a otro nodo o quitarse de la tabla. La modificación de los datos se produce mediante la creación de una o varias extensiones nuevas y el intercambio transaccional de extensiones antiguas con otras nuevas.

Las extensiones contienen una colección de registros que se organizan físicamente en columnas.
Esta técnica se denomina **almacén de columnas**. Permite una codificación y compresión eficientes de los datos, ya que los distintos valores de la misma columna suelen ser similares a los otros. También hace que la consulta de grandes intervalos de datos sea más eficaz, ya que solo es necesario cargar las columnas utilizadas por la consulta. Internamente, cada columna de datos en la extensión se divide en segmentos y los segmentos en bloques. Esta división no se observa en las consultas y permite que Kusto optimice la compresión y la indización de columnas.

Para mantener la eficacia de las consultas, las extensiones más pequeñas se combinan en extensiones más grandes.
La combinación se realiza automáticamente, como un proceso en segundo plano, de acuerdo con la Directiva de [fusión mediante combinación](mergepolicy.md) configurada y la [Directiva de particionamiento](shardingpolicy.md).
La combinación de extensiones reduce la sobrecarga de administración que supone tener un gran número de extensiones para realizar el seguimiento. Lo que es más importante, permite a Kusto optimizar sus índices y mejorar la compresión.

La combinación de extensiones se detiene una vez que una extensión alcanza determinados límites, como el tamaño, desde un punto determinado, la combinación reduce en lugar de aumentar la eficacia.

Cuando se define una [Directiva de partición de datos](partitioningpolicy.md) en una tabla, las extensiones pasan por otro proceso en segundo plano una vez creadas (posterior a la ingesta). Este proceso vuelve a ingerir los datos de las extensiones de origen y crea extensiones *homogéneas* , en las que los valores de la columna que es la *clave de partición* de la tabla pertenecen a la misma partición. Si la Directiva incluye una *clave de partición hash*, todas las extensiones homogéneas que pertenecen a la misma partición se asignarán al mismo nodo de datos del clúster.

> [!NOTE]
> Las operaciones de nivel de extensión, como combinar o modificar etiquetas de extensión, no modifican las extensiones existentes.
> En su lugar, se crean nuevas extensiones en estas operaciones, en función de las extensiones de origen existentes. Las nuevas extensiones reemplazan sus Forefathers en una sola transacción.

El ciclo de vida de extensión común es:

1. La extensión se crea mediante una operación de **ingesta** .
1. La extensión se combina con otras extensiones. Cuando las extensiones que se están combinando son pequeñas, Kusto realmente realiza un proceso de ingesta, denominado **recompilación**. Una vez que las extensiones alcanzan un tamaño determinado, la combinación se realiza solo para los índices. No se modifican los artefactos de datos de las extensiones en el almacenamiento.
1. La extensión combinada (posiblemente una que realiza un seguimiento de su linaje en otras extensiones combinadas, etc.) se quita finalmente debido a una directiva de retención. 
   Cuando se quitan las extensiones, según el tiempo (antigüedad de x horas/días), se usa en el cálculo la fecha de creación de la extensión más reciente dentro de la combinación.

## <a name="extent-creation-time"></a>Hora de creación de la extensión

Una de las partes más importantes de la información para cada extensión es su hora de creación. Esta vez se usa para:

1. Las extensiones de **retención** que se crearon anteriormente se quitarán antes.
1. **Caching** : las extensiones que se crearon recientemente se conservarán en la [memoria caché activa](cachepolicy.md).
1. **Muestreo** : las extensiones recientes se favorecen cuando se usan operaciones de consulta como`take`

De hecho, Kusto realiza un seguimiento de dos `datetime` valores por extensión: `MinCreatedOn` y `MaxCreatedOn` .
Inicialmente, los dos valores son los mismos. Cuando la extensión se combina con otras extensiones, los nuevos valores se ajustan a los valores mínimo y máximo originales de las extensiones combinadas.

Normalmente, el tiempo de creación de una extensión se establece según el tiempo en el que se introducen los datos en la extensión. Los clientes pueden sobrescribir opcionalmente la hora de creación de la extensión, proporcionando un tiempo de creación alternativo en las [propiedades de ingesta](../../ingestion-properties.md).
La sobrescritura es útil, por ejemplo, para fines de retención, si el cliente desea reutilizar los datos y no desea que aparezcan como si llegaran tarde.

## <a name="extent-tagging"></a>Etiquetado de extensiones

Kusto admite la Asociación de varias *etiquetas de extensión* opcional a la extensión, como parte de sus metadatos. Una etiqueta de extensión (o simplemente *etiqueta*) es una cadena que está asociada con la extensión. Puede usar los comandos [. show extents](extents-commands.md#show-extents) para ver las etiquetas asociadas a una extensión y la función [extent-Tags ()](../query/extenttagsfunction.md) para ver las etiquetas asociadas a los registros en una extensión.
Las etiquetas de extensión se pueden usar para describir de forma eficaz las propiedades que son comunes a todos los datos de la extensión.
Por ejemplo, puede Agregar una etiqueta de extensión durante la ingesta, que indica el origen de los datos ingeridos, y utilizar esa etiqueta más adelante. Dado que las extensiones describen los datos, cuando se combinan dos o más combinaciones, sus etiquetas asociadas también se combinan. Las etiquetas de la extensión resultante serán la Unión de todas las etiquetas de esas extensiones combinadas.

Kusto asigna un significado especial a todas las etiquetas de extensión cuyo valor tiene el *sufijo*de *prefijo* de formato, donde el *prefijo* es uno de los siguientes:

* `drop-by:`
* `ingest-by:`

### <a name="drop-by-extent-tags"></a>Etiquetas de extensión ' Drop-by: '

Las etiquetas que comienzan con un `drop-by:` prefijo se pueden usar para controlar las demás extensiones que se van a combinar con. Las extensiones que tienen una `drop-by:` etiqueta determinada se pueden combinar, pero no se combinarán con otras extensiones. Después, puede emitir un comando para quitar extensiones según su `drop-by:` etiqueta.

Por ejemplo:

```kusto
.ingest ... with @'{"tags":"[\"drop-by:2016-02-17\"]"}'

.drop extents <| .show table MyTable extents where tags has "drop-by:2016-02-17" 
```

#### <a name="performance-notes"></a>Notas de rendimiento

* No abusos de `drop-by` etiquetas. La colocación de los datos de la manera mencionada anteriormente está pensada para eventos que raramente ocurren. No se utiliza para reemplazar los datos de nivel de registro y se basa en el hecho de que los datos etiquetados de esta manera son de forma masiva. Si se intenta proporcionar una etiqueta diferente para cada registro o un número pequeño de registros, se podría producir un impacto grave en el rendimiento.
* Si `drop-by` no se necesitan etiquetas durante un período de tiempo después de la ingesta de datos, se recomienda [quitar las etiquetas](extents-commands.md#drop-extent-tags).

### <a name="ingest-by-extent-tags"></a>Etiquetas de extensión ' ingerir por: '

Las etiquetas que comienzan con un `ingest-by:` prefijo se pueden usar para asegurarse de que los datos solo se ingestan una vez. Puede emitir un **`ingestIfNotExists`** comando de propiedad que impida que los datos se ingestan si ya existe una extensión con esta `ingest-by:` etiqueta específica.
Los valores de `tags` y `ingestIfNotExists` son matrices de cadenas que se serializan como JSON.

En el ejemplo siguiente se ingestan datos solo una vez. Los comandos 2 y 3 no hacen nada:

```kusto
.ingest ... with (tags = '["ingest-by:2016-02-17"]')

.ingest ... with (ingestIfNotExists = '["2016-02-17"]')

.ingest ... with (ingestIfNotExists = '["2016-02-17"]', tags = '["ingest-by:2016-02-17"]')
```

> [!NOTE]
> Por lo general, es probable que un comando de introducción incluya una `ingest-by:` etiqueta y una `ingestIfNotExists` propiedad, establecida en el mismo valor, tal y como se muestra en el tercer comando anterior.

#### <a name="performance-notes"></a>Notas de rendimiento

* `ingest-by`No se recomienda el uso de etiquetas using.
Si se sabe que la canalización que alimenta a Kusto tiene duplicados de datos, se recomienda que resuelva estos duplicados lo máximo posible, antes de ingerir los datos en Kusto. Además, use `ingest-by` etiquetas en Kusto solo cuando la parte que se ingesta en Kusto puede introducir duplicados por sí solo (por ejemplo, hay un mecanismo de reintento que se puede superponer con llamadas de ingesta que ya están en curso). Si se intenta establecer una `ingest-by` etiqueta única para cada llamada de ingesta, se podría producir un impacto grave en el rendimiento.
* Si dichas etiquetas no son necesarias durante un período de tiempo después de la ingesta de los datos, se recomienda [quitar las etiquetas](extents-commands.md#drop-extent-tags).
 

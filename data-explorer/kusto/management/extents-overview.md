---
title: 'Extensiones (particiones de datos): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las extensiones (particiones de datos) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/13/2020
ms.openlocfilehash: 839042b50fce6409de29c4cf979a253a7485fb7f
ms.sourcegitcommit: ef009294b386cba909aa56d7bd2275a3e971322f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2020
ms.locfileid: "82977157"
---
# <a name="extents-data-shards"></a>Extensiones (particiones de datos)

## <a name="overview"></a>Información general

Kusto se ha creado para admitir tablas con un gran número de registros (filas) y una gran cantidad de datos. Para poder administrar tablas de gran tamaño, Kusto divide los datos de cada tabla en "tabletas" más pequeñas denominadas **particiones de datos** o **extensiones** (los dos términos son sinónimos), de modo que la Unión de todas las extensiones de la tabla contiene los datos de la tabla. Las extensiones individuales se mantienen más pequeñas que la capacidad de un solo nodo y las extensiones se reparten a través de los nodos del clúster, lo que consigue un escalado horizontal. 

Uno puede pensar en una extensión como un tipo de tabla pequeña. La extensión contiene metadatos (que indican el esquema de los datos en la extensión e información adicional, como la hora de creación y las etiquetas opcionales asociadas a los datos en la extensión) y los datos. Además, la extensión normalmente contiene información que permite a Kusto consultar los datos de forma eficaz, como un índice para cada columna de datos en la extensión, y un diccionario de codificación si se codifican los datos de la columna. Por lo tanto, los datos de la tabla son la Unión de todos los datos de las extensiones de la tabla.

Las extensiones son *inmutables*. Una vez creada, una extensión nunca se modifica y la extensión solo se puede consultar, reasignar a otro nodo o quitar de la tabla. La modificación de datos se produce mediante la creación de una o varias extensiones nuevas y el intercambio transaccional de extensiones antiguas con nuevas extensiones.

Las extensiones contienen una colección de registros, organizados físicamente en columnas.
Esta técnica (denominada **almacén en columnas**) permite codificar y comprimir los datos de forma eficaz (ya que los diferentes valores de la misma columna suelen ser "similares" entre sí) y hacen que las consultas de grandes intervalos de datos sean más eficaces, ya que solo es necesario cargar las columnas utilizadas por la consulta. Internamente, cada columna de datos en la extensión se divide en segmentos y los segmentos en bloques. Esta división (que no se puede observar en las consultas) permite a Kusto optimizar la compresión y la indización de columnas.

Para mantener la eficacia de las consultas, las extensiones más pequeñas se combinan en extensiones más grandes.
Esto lo realiza automáticamente Kusto, como un proceso en segundo plano, de acuerdo con la Directiva de [fusión mediante combinación](mergepolicy.md) configurada y la [Directiva de particionamiento](shardingpolicy.md).
Fusionar las extensiones juntas reduce la sobrecarga de administración que supone tener un gran número de extensiones para realizar el seguimiento, pero lo que es más importante, permite a Kusto optimizar sus índices y mejorar la compresión. La combinación de extensiones se detiene una vez que una extensión alcanza determinados límites, como el tamaño, a menos que se reduzcan algunas extensiones de combinación de puntos, en lugar de aumentar la eficacia.

Cuando se define una [Directiva de partición de datos](partitioningpolicy.md) en una tabla, las extensiones pasan por otro proceso en segundo plano una vez creadas (posterior a la ingesta). Este proceso vuelve a introducir los datos de las extensiones de origen y crea extensiones *homogéneas* , en las que los valores de la columna que es la clave de *partición* de la tabla pertenecen a la misma partición. Si la Directiva incluye una *clave de partición hash*, se garantiza que todas las extensiones homogéneas que pertenecen a la misma partición se asignan al mismo nodo de datos del clúster.

> [!NOTE]
> Las operaciones de nivel de extensión, como la combinación, la modificación de etiquetas de extensión, etc. no modifican las extensiones existentes.
> En su lugar, se crean nuevas extensiones en estas operaciones basadas en las extensiones de origen existentes y, a continuación, estas nuevas extensiones reemplazan sus Forefathers en una sola transacción.

Por lo tanto, el "ciclo de vida" común de una extensión es:

1. La extensión se crea mediante una operación de **ingesta** .
2. La extensión se combina con otras extensiones. Cuando las extensiones que se están combinando son pequeñas, Kusto realmente realiza un proceso de ingesta en ellas (esto se denomina **recompilación**). Una vez que las extensiones alcanzan un tamaño determinado, la combinación se realiza solo para los índices y los artefactos de datos de las extensiones en el almacenamiento no se modifican.
3. La extensión combinada (posiblemente una que realiza un seguimiento de su linaje en otras extensiones combinadas, etc.) se quita finalmente debido a una directiva de retención. Cuando las extensiones se quitan en función del tiempo (antigüedad de x horas/días), la fecha de creación de la extensión más reciente dentro de la combinación se realiza en el cálculo.

## <a name="extent-ingestion-time"></a>Tiempo de ingesta de extensión

Uno de los elementos más importantes de información para cada extensión es su tiempo de ingesta. Kusto usa esta hora para:

1. La retención (extensiones que se han ingerido anteriormente se quitarán antes).
2. El almacenamiento en caché (extensiones que se han ingerido recientemente será más cálido).
3. Muestreo (cuando se usan operaciones de consulta `take`como, se favorecen las extensiones recientes).

De hecho, Kusto realiza un `datetime` seguimiento de dos valores `MinCreatedOn` por `MaxCreatedOn`extensión: y.
Estos valores se inician de la misma forma, pero cuando la extensión se combina con otras extensiones, los valores de la extensión resultante son los valores mínimo y máximo, respectivamente, de todas las extensiones combinadas.

El tiempo de ingesta de una extensión se puede establecer de una de estas tres maneras:

1. Normalmente, el nodo que realiza la ingesta establece este valor en función de su reloj local.
2. Si se establece una **Directiva de tiempo de ingesta** en la tabla, el nodo que realiza la ingesta establece este valor de acuerdo con el reloj local del nodo de administración del clúster, lo que garantiza que todas las ingestas posteriores tendrán un valor de tiempo de ingesta mayor.
3. El cliente puede establecer esta hora. (Esto es útil, por ejemplo, si el cliente desea volver a introducir datos y no desea que los datos que se vuelven a introducir aparezcan como si hubieran llegado tarde, por ejemplo, para fines de retención).    

## <a name="extent-tagging"></a>Etiquetado de extensiones

Como parte de los metadatos almacenados con una extensión, Kusto admite la Asociación de varias *etiquetas de extensión* opcional a la extensión. Una etiqueta de extensión (o simplemente *etiqueta*) es una cadena que está asociada con la extensión. Puede usar los comandos [. show extents](extents-commands.md#show-extents) para ver las etiquetas asociadas a una extensión y la función [extent-Tags ()](../query/extenttagsfunction.md) para ver las etiquetas asociadas a los registros en una extensión.
Las etiquetas de extensión se pueden usar para describir de forma eficaz las propiedades que pertenecen a todos los datos de la extensión.
Por ejemplo, puede Agregar una etiqueta de extensión durante la ingesta que indica el origen de los datos que se van a ingerir y, posteriormente, usar esa etiqueta. A medida que describen los datos, cuando se combinan dos o más extensiones, las etiquetas asociadas también se combinan al tener las etiquetas de la extensión resultante en la Unión de todas las etiquetas de extensión de las extensiones que se están combinando.

Kusto asigna un significado especial a todas las etiquetas de extensión cuyo valor tiene el *sufijo*de *prefijo* de formato, donde el *prefijo* es uno de los siguientes:

* `drop-by:`
* `ingest-by:`

### <a name="drop-by-extent-tags"></a>Etiquetas de extensión ' Drop-by: '

Las etiquetas que comienzan con **`drop-by:`** un prefijo se pueden usar para controlar las demás extensiones que se van a combinar con; las extensiones que tienen una etiqueta `drop-by:` determinada se pueden combinar, pero no se combinarán con otras extensiones. Esto permite al usuario emitir un comando para quitar extensiones según su `drop-by:` etiqueta, como el comando siguiente:

```kusto
.ingest ... with @'{"tags":"[\"drop-by:2016-02-17\"]"}'

.drop extents <| .show table MyTable extents where tags has "drop-by:2016-02-17" 
```

#### <a name="performance-notes"></a>Notas de rendimiento

* No se recomienda `drop-by` usar etiquetas con exceso de uso. La compatibilidad con la eliminación de datos de la manera mencionada anteriormente está pensada para eventos con poca frecuencia, no es para reemplazar datos de nivel de registro y, de hecho, se basa en el hecho de que los datos que se etiquetan de esta manera son "voluminosos". Si se intenta proporcionar una etiqueta diferente para cada registro o un número pequeño de registros, se podría producir un impacto grave en el rendimiento.
* En los casos en los que estas etiquetas no son necesarias un período de tiempo después de la ingesta de datos, se recomienda [quitar las etiquetas](extents-commands.md#drop-extent-tags).

### <a name="ingest-by-extent-tags"></a>Etiquetas de extensión ' ingerir por: '

Las etiquetas que comienzan con **`ingest-by:`** un prefijo se pueden usar para asegurarse de que los datos solo se ingestan una vez. El usuario puede emitir un comando de introducción que evita que los datos se ingestan si ya existe una extensión con esta `ingest-by:` etiqueta específica mediante la **`ingestIfNotExists`** propiedad.
Los valores de `tags` y `ingestIfNotExists` son matrices de cadenas que se serializan como JSON.

En el ejemplo siguiente se ingestan datos solo una vez (los comandos 2 y 3 no hacen nada):

```kusto
.ingest ... with (tags = '["ingest-by:2016-02-17"]')

.ingest ... with (ingestIfNotExists = '["2016-02-17"]')

.ingest ... with (ingestIfNotExists = '["2016-02-17"]', tags = '["ingest-by:2016-02-17"]')
```

> [!NOTE]
> En el caso común, es probable que un comando de introducción incluya una `ingest-by:` etiqueta y una `ingestIfNotExists` propiedad, que se establezcan en el mismo valor (como se muestra en el tercer comando anterior).

#### <a name="performance-notes"></a>Notas de rendimiento

- No se `ingest-by` recomienda usar etiquetas de sobreutilización.
Si se sabe que la canalización que alimenta a Kusto tiene duplicados de datos, se recomienda resolverlos tanto como sea posible antes de ingerir los datos en Kusto `ingest-by` , y usar etiquetas en Kusto solo para los casos en los que la parte que ingesta en Kusto pueda introducir duplicados por sí solo (por ejemplo, hay un mecanismo de reintento que se puede superponer con llamadas de ingesta Si se intenta establecer una etiqueta `ingest-by` única para cada llamada de ingesta, se podría producir un impacto grave en el rendimiento.
- En los casos en los que estas etiquetas no son necesarias un período de tiempo después de la ingesta de datos, se recomienda [quitar las etiquetas](extents-commands.md#drop-extent-tags).

---
title: 'Extensiones (particiones de datos): Explorador de azure Data Explorer ( Extensión) Microsoft Docs'
description: En este artículo se describen las extensiones (particiones de datos) en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/13/2020
ms.openlocfilehash: 16afb2dc7d2310e9e63ec3465937ac84c96b27e4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521019"
---
# <a name="extents-data-shards"></a>Extensiones (particiones de datos)

## <a name="overview"></a>Información general

Kusto está diseñado para admitir tablas con un gran número de registros (filas) y una gran cantidad de datos. Para poder manejar tablas tan grandes, Kusto divide los datos de cada tabla en "tabletas" más pequeñas **llamadas fragmentos** de datos o **extensiones** (los dos términos son sinónimos), de modo que la unión de todas las extensiones de la tabla contiene los datos de la tabla. A continuación, las extensiones individuales se mantienen más pequeñas que la capacidad de un solo nodo y las extensiones se distribuyen entre los nodos del clúster, lo que logra un escalado horizontal. 

Uno puede pensar en una extensión como una especie de mini-mesa. La extensión contiene metadatos (que indican el esquema de los datos en la extensión y información adicional, como su tiempo de creación y etiquetas opcionales asociadas a los datos en la extensión) y los datos. Además, la extensión normalmente contiene información que permite a Kusto consultar los datos de forma eficaz, como un índice para cada columna de datos de la extensión, y un diccionario de codificación si se codifican los datos de columna. Por lo tanto, los datos de la tabla son la unión de todos los datos en las extensiones de la tabla.

Las extensiones son *inmutables*. Una vez creada, una extensión nunca se modifica y la extensión solo se puede consultar, reasignar a un nodo diferente o quitarla de la tabla. La modificación de datos se produce mediante la creación de una o más extensiones nuevas y el intercambio transaccional de extensiones antiguas con nuevas extensiones.

Las extensiones contienen una colección de registros, organizados físicamente en columnas.
Esta técnica (denominada **almacén en columnas)** permite codificar y comprimir los datos de forma eficaz (porque diferentes valores de la misma columna a menudo se "parecen" entre sí) y hace que las consultas de grandes intervalos de datos sean más eficaces porque solo es necesario cargar las columnas utilizadas por la consulta. Internamente, cada columna de datos de la extensión se subdivide en segmentos y los segmentos en bloques. Esta división (que no se puede observar para las consultas) permite a Kusto optimizar la compresión e indexación de columnas.

Para mantener la eficacia de las consultas, las extensiones más pequeñas se combinan en extensiones más grandes.
Esto lo realiza automáticamente Kusto, como un proceso en segundo plano, de acuerdo con la directiva de [combinación](mergepolicy.md) configurada y la directiva de [particionamiento.](shardingpolicy.md)
La combinación de extensiones reduce la sobrecarga de gestión de tener un gran número de extensiones para realizar un seguimiento, pero lo que es más importante, permite a Kusto optimizar sus índices y mejorar la compresión. La fusión de extensión se detiene una vez que una extensión alcanza ciertos límites, como el tamaño, ya que más allá de un determinado punto se reduce en lugar de aumentar la eficiencia.

Cuando se define una directiva de [partición](partitioningpolicy.md) de datos en una tabla, las extensiones pasan por otro proceso en segundo plano después de crearlos (después de la ingesta). Este proceso vuelve a ingerir los datos de las extensiones de origen y crea extensiones *homogéneas,* en las que los valores de la columna que es la clave de *partición* de la tabla pertenecen a la misma partición. Si la directiva incluye una clave de *partición hash,* se garantiza que todas las extensiones homogéneas que pertenecen a la misma partición se asignan al mismo nodo de datos del clúster.

> [!NOTE]
> Las operaciones de nivel de extensión, como la combinación, la modificación de etiquetas de extensión, etc. no modifican las extensiones existentes.
> Más bien, se crean nuevas extensiones en estas operaciones basadas en extensiones de origen existentes y, a continuación, estas nuevas extensiones reemplazan a sus antepasados en una sola transacción.

Por lo tanto, el "ciclo de vida" común de una extensión es:

1. La extensión se crea mediante una operación de **ingesta.**
2. La extensión se combina con otras extensiones. Cuando las extensiones que se fusionan son pequeñas, Kusto realmente realiza un proceso de ingesta en ellos (esto se denomina **reconstrucción).** Una vez que las extensiones alcanzan un tamaño determinado, la combinación se realiza solo para los índices y los artefactos de datos de las extensiones en el almacenamiento no se modifican.
3. La extensión combinada (posiblemente una que realiza un seguimiento de su linaje a otras extensiones combinadas, etc.) se descarta finalmente debido a una directiva de retención. Cuando las extensiones se descartan en función del tiempo (más antiguos x horas / días) la fecha de creación de la extensión más reciente dentro de la combinada se toma en el cálculo.

## <a name="extent-ingestion-time"></a>Tiempo de ingestión de extensión

Una de las piezas de información más importantes para cada extensión es su tiempo de ingestión. Este tiempo es utilizado por Kusto para:

1. Retención (las extensiones que se ingerieron anteriormente se eliminarán antes).
2. Almacenamiento en caché (las extensiones que se han ingerido recientemente será más caliente).
3. Muestreo (cuando se utilizan `take`operaciones de consulta como , se favorecen las extensiones recientes).

De hecho, Kusto `datetime` realiza un `MinCreatedOn` `MaxCreatedOn`seguimiento de dos valores por extensión: y .
Estos valores comienzan igual, pero cuando la extensión se combina con otras extensiones, los valores de la extensión resultante son los valores mínimo y máximo, respectivamente, sobre todas las extensiones combinadas.

El tiempo de ingesta de una extensión puede establecerse de tres maneras:

1. Normalmente, el nodo que realiza la ingestión establece este valor según su reloj local.
2. Si se establece una directiva de tiempo de **ingesta** en la tabla, el nodo que realiza la ingesta establece este valor según el reloj local del nodo de administración del clúster, lo que garantiza que todas las ingestas posteriores tendrán un valor de tiempo de ingesta más alto.
3. El cliente puede establecer esta hora. (Esto es útil, por ejemplo, si el cliente desea reing datos y no desea que los datos reingerdos aparezcan como si hubieran llegado tarde, por ejemplo, para fines de retención).    

## <a name="extent-tagging"></a>Etiquetado de extensión

Como parte de los metadatos almacenados con una extensión, Kusto admite la asociación de varias etiquetas de *extensión* opcionales en la medida. Una etiqueta de extensión (o simplemente *etiqueta),* es una cadena asociada a la extensión. Se pueden utilizar los [comandos .show extents](extents-commands.md#show-extents) para ver las etiquetas asociadas a una extensión y la función [extent-tags()](../query/extenttagsfunction.md) para ver las etiquetas asociadas a los registros en una extensión.
Las etiquetas de extensión se pueden utilizar para describir eficazmente las propiedades que pertenecen a todos los datos de la extensión.
Por ejemplo, se podría agregar una etiqueta de extensión durante la ingesta que indique el origen de los datos que se ingiren y, más adelante, usar esa etiqueta. A medida que describen los datos, cuando se combinan dos o más extensiones, sus etiquetas asociadas también se combinan haciendo que las etiquetas de la extensión resultante sean la unión de todas las etiquetas de extensión de las extensiones que se fusionan.

Kusto asigna un significado especial a todas las etiquetas de extensión cuyo valor tiene el *sufijo*de prefijo de *formato,* donde *prefix* es uno de:

* `drop-by:`
* `ingest-by:`

## <a name="drop-by-extent-tags"></a>etiquetas de extensión 'drop-by:'

Las etiquetas que **`drop-by:`** comienzan con un prefijo se pueden utilizar para controlar con qué otras extensiones combinar; las extensiones que `drop-by:` tienen una etiqueta determinada se pueden combinar, pero no se fusionarán con otras extensiones. Esto permite al usuario emitir un comando para `drop-by:` quitar extensiones según su etiqueta, como el siguiente comando:

```kusto
.ingest ... with @'{"tags":"[\"drop-by:2016-02-17\"]"}'

.drop extents <| .show table MyTable extents where tags has "drop-by:2016-02-17" 
```

### <a name="performance-notes"></a>Notas de rendimiento

* No se `drop-by` recomienda el uso excesivo de etiquetas. El soporte para descartar datos de la manera mencionada anteriormente está destinado a eventos que rara vez ocurren, no es para reemplazar datos de nivel de registro y se basa críticamente en el hecho de que los datos que se etiquetan de esta manera son "abultados". Intentar dar una etiqueta diferente para cada registro, o un pequeño número de registros, podría resultar con un impacto grave en el rendimiento.
* En los casos en que estas etiquetas no sean necesarias algún período de tiempo después de la ingesta de datos, se recomienda [quitar las etiquetas](extents-commands.md#drop-extent-tags).

## <a name="ingest-by-extent-tags"></a>etiquetas de extensión 'ingest-by:'

Las etiquetas que **`ingest-by:`** comienzan con un prefijo se pueden utilizar para garantizar que los datos solo se ingiren una vez. El usuario puede emitir un comando ingest que impide que se ingiren `ingest-by:` los datos **`ingestIfNotExists`** si ya hay una extensión con esta etiqueta específica mediante la propiedad.
Los valores `tags` de `ingestIfNotExists` ambas matrices y matrices de cadenas, serializadas como JSON.

En el ejemplo siguiente se ingienden datos solo una vez (los comandos 2o y 3o no harán nada):

```kusto
.ingest ... with (tags = '["ingest-by:2016-02-17"]')

.ingest ... with (ingestIfNotExists = '["2016-02-17"]')

.ingest ... with (ingestIfNotExists = '["2016-02-17"]', tags = '["ingest-by:2016-02-17"]')
```

> [!NOTE]
> En el caso común, es probable que `ingest-by:` un comando `ingestIfNotExists` de ingesta incluya una etiqueta y una propiedad, establecidas en el mismo valor (como se muestra en el comando 3er anterior).

### <a name="performance-notes"></a>Notas de rendimiento

- No `ingest-by` se recomienda el uso excesivo de etiquetas.
Si se sabe que la canalización que alimenta a Kusto tiene duplicaciones de datos, se recomienda resolverlas `ingest-by` tanto como sea posible antes de ingerir los datos en Kusto, y usar etiquetas en Kusto solo para los casos en los que la parte que ingesta a Kusto podría introducir duplicados por sí misma (por ejemplo, hay un mecanismo de reintento que puede superponerse con las llamadas de ingestión ya en curso). Intentar establecer una `ingest-by` etiqueta única para cada llamada de ingesta podría resultar con un impacto grave en el rendimiento.
- En los casos en que estas etiquetas no sean necesarias algún período de tiempo después de la ingesta de datos, se recomienda [quitar las etiquetas](extents-commands.md#drop-extent-tags).
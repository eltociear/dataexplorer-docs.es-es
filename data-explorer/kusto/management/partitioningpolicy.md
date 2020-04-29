---
title: 'Directiva de creación de particiones de datos (versión preliminar): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la Directiva de creación de particiones de datos (versión preliminar) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: ad255c6930e76628a5187fa8d321e3445dbb5f99
ms.sourcegitcommit: fbe298e88542c0dcea0f491bb53ac427f850f729
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/24/2020
ms.locfileid: "82138873"
---
# <a name="data-partitioning-policy-preview"></a>Directiva de particionamiento de datos (versión preliminar)

La Directiva de particionamiento define si se deben particionar las extensiones (particiones de [datos)](../management/extents-overview.md) y para una tabla específica.

> [!NOTE]
> La característica de creación de particiones de datos está en *versión preliminar*.

El propósito principal de la Directiva es mejorar el rendimiento de las consultas que se sabe que se van a restringir a un pequeño subconjunto de valores de las columnas con particiones.
Una ventaja potencial secundaria es la compresión mejor de los datos.

> [!WARNING]
> Aunque no hay ningún límite codificado de forma rígida en la cantidad de tablas que pueden tener la Directiva definida, cada tabla adicional agrega sobrecarga al proceso de creación de particiones de datos en segundo plano que se ejecuta en los nodos del clúster y puede requerir recursos adicionales del clúster; consulte [Capacity (capacidad](#capacity)).

## <a name="partition-keys"></a>Claves de partición

Se admiten los siguientes tipos de claves de partición:

|Clase                                                   |Tipo de columna |Propiedades de la partición                    |Valor de partición                                        |
|-------------------------------------------------------|------------|----------------------------------------|-------------------------------------------------------|
|[Hash](#hash-partition-key)                            |`string`    |`Function`, `MaxPartitionCount`, `Seed` | `Function`(`ColumnName`, `MaxPartitionCount`, `Seed`) |
|[Intervalo uniforme](#uniform-range-datetime-partition-key) |`datetime`  |`RangeSize`, `Reference`                | `bin_at`(`ColumnName`, `RangeSize`, `Reference`)      |

### <a name="hash-partition-key"></a>Clave de partición hash

Aplicar una clave de partición hash en `string`una columna con tipo en una tabla es adecuado cuando la *mayoría* de las consultas utiliza filtros de`==`igualdad `in()`(,) en `string`una columna de tipo específico de *dimensión grande*, `application_ID`como `tenant_ID` o. `user_ID`

* Se utiliza una función de módulo hash para particionar los datos.
* Todas las extensiones *homogéneas* (con particiones) que pertenecen a la misma partición se asignan al mismo nodo de datos.
* Los datos en extensiones *homogéneas* (con particiones) se ordenan por la clave de partición hash.
  * No es necesario incluir la clave de partición en una [Directiva de orden de filas](roworderpolicy.md), si se define una en la tabla.
* Las consultas que utilizan la [estrategia de orden aleatorio](../query/shufflequery.md)y en `shuffle key` las que `join`se `summarize` usa `make-series` en, o es la clave de partición hash de la tabla, se espera que funcionen mejor, ya que la cantidad de datos necesaria para moverse entre los nodos del clúster se reduce significativamente.

#### <a name="partition-properties"></a>Propiedades de la partición

* `Function`es el nombre de una función de módulo hash que se va a utilizar.
  * Valor admitido `XxHash64`:.
* `MaxPartitionCount`es el número máximo de particiones que se van a crear (el argumento de módulo para la función de módulo hash) por período de tiempo.
  * Se admiten valores en el `(1,1024]`intervalo.
    * Se espera que el valor sea:
      * Mayor que el número de nodos del clúster
      * Menor que la cardinalidad de la columna.
    * Cuanto mayor sea el valor, mayor será la sobrecarga del proceso de creación de particiones de datos en los nodos del clúster y mayor será el número de extensiones para cada período de tiempo.
    * Un valor recomendado para empezar es `256`.
      * Si es necesario, se puede ajustar (normalmente en adelante), en función de las consideraciones mencionadas anteriormente y/o en función de la medición de la ventaja en el rendimiento de las consultas frente a la sobrecarga de la creación de particiones de los datos después de la ingesta.
* `Seed`es el valor que se va a usar para aleatorizar el valor hash.
  * El valor debe ser un entero positivo.
  * El valor recomendado (y predeterminado, si no se especifica) `1`es.

#### <a name="example"></a>Ejemplo

A continuación se indica una clave de partición hash `string`en una columna con tipo `tenant_id`denominada.

`XxHash64` Usa la función hash, con un `MaxPartitionCount` de `256`y el valor predeterminado `Seed` de. `1`

```json
{
  "ColumnName": "tenant_id",
  "Kind": "Hash",
  "Properties": {
    "Function": "XxHash64",
    "MaxPartitionCount": 256,
    "Seed": 1
  }
}
```

### <a name="uniform-range-datetime-partition-key"></a>Clave de partición DateTime de intervalo uniforme

Aplicar una clave de partición DateTime de intervalo uniforme `datetime`en una columna con tipo en una tabla es adecuado cuando no es *probable* que los datos ingeridos en la tabla se ordenen según esta columna. En tales casos, puede ser útil volver a ordenar los datos entre las extensiones para que cada extensión finalice, incluidos los registros de un intervalo de tiempo limitado, lo que resultará en `datetime` que los filtros de la columna en el momento de la consulta sean más eficaces.

* La función de partición usada es [bin_at ()](../query/binatfunction.md) y no es personalizable.

#### <a name="partition-properties"></a>Propiedades de la partición

* `RangeSize`es una `timespan` constante escalar que indica el tamaño de cada partición de fecha y hora.
* `Reference`es una `datetime` constante escalar de tipo que indica un punto fijo en el tiempo según el cual se alinean las particiones DateTime.
  * Si hay registros en los que la clave de partición DateTime `null` tiene valores, su valor de partición se establece en el `Reference`valor de.

#### <a name="example"></a>Ejemplo

A continuación se indica una clave de partición de intervalo DateTime `datetime`uniforme en una columna con tipo denominada`timestamp`

Usa `datetime(1970-01-01)` como punto de referencia, con un tamaño de `1d` para cada partición.

```json
{
  "ColumnName": "timestamp",
  "Kind": "UniformRange",
  "Properties": {
    "Reference": "1970-01-01T00:00:00",
    "RangeSize": "1.00:00:00"
  }
}
```

## <a name="the-policy-object"></a>El objeto de Directiva

De forma predeterminada, la Directiva de creación de particiones `null`de datos de una tabla es, en cuyo caso no se crearán particiones en los datos de la tabla.

La Directiva de creación de particiones de datos tiene las siguientes propiedades principales:

* **PartitionKeys**:
  * Colección de [claves de partición](#partition-keys) que definen cómo crear particiones de los datos en la tabla.
  * Una tabla puede tener hasta una `2` clave de partición, con una de las tres opciones siguientes:
    * 1 [clave de partición hash](#hash-partition-key).
    * 1 [clave de partición DateTime de intervalo uniforme](#uniform-range-datetime-partition-key).
    * 1 [clave de partición hash](#hash-partition-key) y 1 [clave de partición DateTime de intervalo uniforme](#uniform-range-datetime-partition-key).
  * Cada clave de partición tiene las siguientes propiedades:
    * `ColumnName`: `string ` -El nombre de la columna según la cual se crearán las particiones de los datos.
    * `Kind`: `string` -El tipo de partición de datos que se`Hash` va `UniformRange`a aplicar (o).
    * `Properties`: `property bag` define los parámetros en función de la partición que se realiza.

* **EffectiveDateTime**:
  * Fecha y hora UTC desde la que se aplica la Directiva.
  * Esta propiedad es *opcional* ; si no se especifica, la Directiva surtirá efecto en los datos ingeridos después de aplicar la Directiva.
  * El proceso de creación de particiones no tiene en cuenta las extensiones no homogéneas (sin particiones) que estén enlazadas para que se quiten pronto debido a la retención (es decir, el tiempo de creación precede al 90% del período efectivo de eliminación temporal de la tabla).

### <a name="example"></a>Ejemplo

A continuación se proporciona un objeto de directiva de particionamiento de datos con dos claves de partición:
1. Una clave de partición hash en `string`una columna con tipo denominada `tenant_id`.
    - Usa la `XxHash64` función hash, con un `MaxPartitionCount` de 256 y el valor predeterminado `Seed` de. `1`
2. Una clave de partición de intervalo DateTime uniforme `datetime`en una columna con tipo denominada`timestamp`
    - Usa `datetime(1970-01-01)` como punto de referencia, con un tamaño de `1d` para cada partición.

```json
{
  "PartitionKeys": [
    {
      "ColumnName": "tenant_id",
      "Kind": "Hash",
      "Properties": {
        "Function": "XxHash64",
        "MaxPartitionCount": 256,
        "Seed": 1
      }
    },
    {
      "ColumnName": "timestamp",
      "Kind": "UniformRange",
      "Properties": {
        "Reference": "1970-01-01T00:00:00",
        "RangeSize": "1.00:00:00"
      }
    }
  ]
}
```

### <a name="additional-properties"></a>Propiedades adicionales

Las siguientes propiedades se pueden definir como parte de la Directiva, pero son *opcionales* y no se recomienda cambiarlas.

* **MinRowCountPerOperation**:
  * Destino mínimo de la suma del recuento de filas de las extensiones de origen de una sola operación de particionamiento de datos.
  * Esta propiedad es *opcional*y su valor predeterminado es `0`.

* **MaxRowCountPerOperation**:
  * Destino máximo para la suma del recuento de filas de las extensiones de origen de una sola operación de particionamiento de datos.
  * Esta propiedad es *opcional*, y su valor predeterminado es `0` (en cuyo caso, se aplica un destino predeterminado de 5 millones registros).

## <a name="notes"></a>Notas

### <a name="the-data-partitioning-process"></a>Proceso de creación de particiones de datos

* La creación de particiones de datos se ejecuta como un proceso en segundo plano posterior a la ingesta en el clúster.
  * Se espera que una tabla que se está ingesta continuamente en tenga siempre un "final" de datos que todavía se van a particionar (extensiones*no homogéneas* ).
* La creación de particiones de datos solo se ejecuta en extensiones activas, independientemente del `EffectiveDateTime` valor de la propiedad en la Directiva.
  * Si se requiere la creación de particiones en frío, debe ajustar temporalmente la [Directiva de almacenamiento en caché en](cachepolicy.md) consecuencia.

#### <a name="monitoring"></a>Supervisión

* Puede supervisar el progreso y el estado de la creación de particiones en un clúster mediante el comando [. show Diagnostics](../management/diagnostics.md#show-diagnostics) :

```kusto
.show diagnostics
| project MinPartitioningPercentageInSingleTable,
          TableWithMinPartitioningPercentage
```

La salida incluye:

  * `MinPartitioningPercentageInSingleTable`: el porcentaje mínimo de datos particionados en todas las tablas que tienen una directiva de particionamiento de datos en el clúster.
      * Si este porcentaje se mantiene constantemente en el 90%, evalúe la capacidad de particionamiento del clúster (consulte [a continuación](partitioningpolicy.md#capacity)).
  * `TableWithMinPartitioningPercentage`: el nombre completo de la tabla cuyo porcentaje de particionamiento se muestra arriba.

#### <a name="capacity"></a>Capacity

* Como el proceso de creación de particiones de datos da como resultado la creación de más extensiones, podría ser necesario (gradualmente y linealmente) aumentar la [capacidad de combinación](../management/capacitypolicy.md#extents-merge-capacity) de las extensiones del clúster para que el proceso de [combinación de extensiones](../management/extents-overview.md) sea capaz de mantenerse al día.
* Si es necesario (por ejemplo, en el caso de un rendimiento de ingesta alto y/o un número de tablas suficientemente grande que requieran particiones), la [capacidad de partición](../management/capacitypolicy.md#extents-partition-capacity) del clúster se puede aumentar (gradualmente y linealmente) para permitir la ejecución de un número mayor de operaciones de particionamiento simultáneo.
  * En caso de que el aumento de la creación de particiones provoque un aumento significativo del uso de los recursos del clúster, escale el clúster vertical u horizontalmente, ya sea manualmente o habilitando el escalado automático.

### <a name="outliers-in-partitioned-columns"></a>Valores atípicos en columnas con particiones

* Si una clave de partición hash tiene un porcentaje suficientemente grande de valores que no se rellenan correctamente (por ejemplo, están vacíos o tienen un valor genérico), esto podría contribuir a tener una distribución sin equilibrio de datos entre los nodos del clúster.
* Si una clave de partición DateTime de intervalo uniforme tiene un porcentaje suficientemente grande de valores que están "lejos" de la mayoría de los valores de la columna (por ejemplo, valores de fecha y hora desde el pasado o el futuro).

En ambos casos, debe "corregir" los datos o filtrar los registros irrelevantes de los datos antes o en el momento de la ingesta (por ejemplo, mediante una [Directiva de actualización](updatepolicy.md)) para reducir la sobrecarga de la creación de particiones de datos en el clúster.

## <a name="next-steps"></a>Pasos siguientes

Use los [comandos de control de directivas de partición](../management/partitioning-policy.md) para administrar las directivas de particionamiento de datos para las tablas.
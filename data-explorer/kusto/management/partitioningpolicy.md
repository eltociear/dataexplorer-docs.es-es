---
title: 'Directiva de creación de particiones de datos (versión preliminar): Azure Explorador de datos'
description: En este artículo se describe la Directiva de creación de particiones de datos (versión preliminar) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/10/2020
ms.openlocfilehash: 768f07307a6f43c2af2db79bc1221c140b7c9a6f
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84664983"
---
# <a name="data-partitioning-policy"></a>Directiva de particionamiento de datos

La Directiva de particionamiento define si se deben particionar las extensiones (particiones de [datos)](../management/extents-overview.md) y para una tabla específica.

El propósito principal de la Directiva es mejorar el rendimiento de las consultas que se sabe que restringen el conjunto de datos de los valores de las columnas con particiones, o agregar o combinar en una columna de cadena de cardinalidad alta. La Directiva también puede mejorar la compresión de los datos.

> [!CAUTION]
> No hay ningún límite codificado de forma rígida establecido en el número de tablas que pueden tener definida la Directiva. Sin embargo, cada tabla adicional agrega sobrecarga al proceso de creación de particiones de datos en segundo plano que se ejecuta en los nodos del clúster. Puede dar lugar a que se usen más recursos de clúster. Para obtener más información, consulte [Capacity (capacidad](#capacity)).

## <a name="partition-keys"></a>Claves de partición

Se admiten los siguientes tipos de claves de partición.

|Tipo                                                   |Tipo de columna |Propiedades de la partición                    |Valor de partición                                        |
|-------------------------------------------------------|------------|----------------------------------------|----------------------|
|[Hash](#hash-partition-key)                            |`string`    |`Function`, `MaxPartitionCount`, `Seed` | `Function`(`ColumnName`, `MaxPartitionCount`, `Seed`) |
|[Intervalo uniforme](#uniform-range-datetime-partition-key) |`datetime`  |`RangeSize`, `Reference`                | `bin_at`(`ColumnName`, `RangeSize`, `Reference`)      |

### <a name="hash-partition-key"></a>Clave de partición hash

Aplicar una clave de partición hash en una `string` columna de tipo de una tabla es adecuado cuando la mayoría de las consultas utiliza filtros de igualdad ( `==` , `in()` ) o cuando se agregan o se combinan en una `string` columna de tipo específico de *dimensión grande* (cardinalidad de 10 millones o más), como `application_ID` , `tenant_ID` o `user_ID` .

* Se utiliza una función de módulo hash para particionar los datos.
* Todas las extensiones homogéneas (con particiones) que pertenecen a la misma partición se asignan al mismo nodo de datos.
* Los datos en extensiones homogéneas (con particiones) se ordenan por la clave de partición hash.
  * No es necesario incluir la clave de partición hash en la [Directiva de orden de filas](roworderpolicy.md), si se ha definido una en la tabla.
* Las consultas que utilizan la [estrategia de orden aleatorio](../query/shufflequery.md)y en las que `shuffle key` se usa en `join` , `summarize` o `make-series` es la clave de partición hash de la tabla, se espera que funcionen mejor, ya que la cantidad de datos necesaria para moverse entre los nodos del clúster se reduce significativamente.

#### <a name="partition-properties"></a>Propiedades de la partición

* `Function`es el nombre de una función de módulo hash que se va a utilizar.
  * Valor admitido: `XxHash64` .
* `MaxPartitionCount`es el número máximo de particiones que se van a crear (el argumento de módulo para la función de módulo hash) por período de tiempo.
  * Los valores admitidos están en el intervalo `(1,1024]` .
    * Se espera que el valor sea:
      * Mayor que el número de nodos del clúster
      * Menor que la cardinalidad de la columna.
    * Cuanto mayor sea el valor, mayor será la sobrecarga del proceso de creación de particiones de datos en los nodos del clúster y mayor será el número de extensiones para cada período de tiempo.
    * Se recomienda comenzar con un valor de `256` .
      * Ajuste el valor según sea necesario, en función de las consideraciones anteriores, o en función de las ventajas del rendimiento de las consultas frente a la sobrecarga que supone la creación de particiones de los datos después de la ingesta.
* `Seed`es el valor que se va a usar para aleatorizar el valor hash.
  * El valor debe ser un entero positivo.
  * El valor recomendado es `1` , que es el valor predeterminado, si no se especifica.

#### <a name="example"></a>Ejemplo

Una clave de partición hash en una `string` columna con tipo denominada `tenant_id` .
Usa la `XxHash64` función hash, con un `MaxPartitionCount` de `256` y el valor predeterminado `Seed` de `1` .

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

Aplicar una clave de partición DateTime de intervalo uniforme en una `datetime` columna con tipo en una tabla es adecuado cuando no es probable que los datos ingeridos en la tabla se ordenen según esta columna. Puede ser útil reorganizar los datos entre las extensiones para que cada extensión finalice, incluidos los registros de un intervalo de tiempo limitado. Reconstruir da como resultado que los filtros de la `datetime` columna sean más eficaces en el momento de la consulta.

* La función de partición usada es [bin_at ()](../query/binatfunction.md) y no es personalizable.

#### <a name="partition-properties"></a>Propiedades de la partición

* `RangeSize`es una `timespan` constante escalar que indica el tamaño de cada partición de fecha y hora.
  Se recomienda:
  * Comience con el valor `1.00:00:00` (un día).
  * No establezca un valor más corto, ya que puede dar lugar a que la tabla tenga un gran número de extensiones pequeñas que no se pueden combinar.
* `Reference`es una `datetime` constante escalar que indica un punto fijo en el tiempo, en función de las particiones de fecha y hora que se alinean.
  * Se recomienda comenzar por `1970-01-01 00:00:00` .
  * Si hay registros en los que la clave de partición DateTime tiene `null` valores, su valor de partición se establece en el valor de `Reference` .

#### <a name="example"></a>Ejemplo

El fragmento de código muestra una clave de partición de intervalo DateTime uniforme en una `datetime` columna con tipo denominada `timestamp` .
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

De forma predeterminada, la Directiva de creación de particiones de datos de una tabla es `null` , en cuyo caso no se crearán particiones en los datos de la tabla.

La Directiva de creación de particiones de datos tiene las siguientes propiedades principales:

* **PartitionKeys**:
  * Colección de [claves de partición](#partition-keys) que definen cómo crear particiones de los datos en la tabla.
  * Una tabla puede tener hasta `2` una clave de partición, con una de las siguientes opciones.
    * Una [clave de partición hash](#hash-partition-key).
    * Una [clave de partición DateTime de intervalo uniforme](#uniform-range-datetime-partition-key).
    * Una [clave de partición hash](#hash-partition-key) y una [clave de partición DateTime de intervalo uniforme](#uniform-range-datetime-partition-key).
  * Cada clave de partición tiene las siguientes propiedades:
    * `ColumnName`: `string ` -El nombre de la columna según la cual se crearán las particiones de los datos.
    * `Kind`: `string` -El tipo de partición de datos que se va a aplicar ( `Hash` o `UniformRange` ).
    * `Properties`: `property bag` Define los parámetros en función de la partición que se realiza.

* **EffectiveDateTime**:
  * Fecha y hora UTC desde la que se aplica la Directiva.
  * Esta propiedad es opcional. Si no se especifica, la Directiva surtirá efecto en los datos ingeridos después de aplicar la Directiva.
  * El proceso de creación de particiones omite las extensiones no homogéneas (sin particiones) que se pueden quitar debido a la retención, porque su hora de creación precede al 90% del período efectivo de eliminación temporal de la tabla.

### <a name="example"></a>Ejemplo

Objeto de directiva de particionamiento de datos con dos claves de partición.
1. Una clave de partición hash en una `string` columna con tipo denominada `tenant_id` .
    - Usa la `XxHash64` función hash, con un `MaxPartitionCount` de 256 y el valor predeterminado `Seed` de `1` .
1. Una clave de partición de intervalo DateTime uniforme en una `datetime` columna de tipo denominada `timestamp` .
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

Las siguientes propiedades se pueden definir como parte de la Directiva, pero son opcionales y se recomienda que no las cambie.

* **MinRowCountPerOperation**:
  * Destino mínimo de la suma del recuento de filas de las extensiones de origen de una sola operación de particionamiento de datos.
  * Esta propiedad es opcional. Su valor predeterminado es `0`.

* **MaxRowCountPerOperation**:
  * Destino máximo para la suma del recuento de filas de las extensiones de origen de una sola operación de particionamiento de datos.
  * Esta propiedad es opcional. Su valor predeterminado es `0` , con un destino predeterminado de 5 millones registros.

## <a name="notes"></a>Notas

### <a name="the-data-partitioning-process"></a>Proceso de creación de particiones de datos

* La creación de particiones de datos se ejecuta como un proceso en segundo plano posterior a la ingesta en el clúster.
  * Se espera que una tabla que se ingesta continuamente en tenga siempre el "final" de los datos que todavía se van a particionar (extensiones no homogéneas).
* La creación de particiones de datos solo se ejecuta en extensiones activas, independientemente del valor de la `EffectiveDateTime` propiedad en la Directiva.
  * Si se requiere la creación de particiones en frío, tendrá que ajustar temporalmente la [Directiva de almacenamiento en caché](cachepolicy.md).

#### <a name="monitoring"></a>Supervisión

Use el comando [. show Diagnostics](../management/diagnostics.md#show-diagnostics) para supervisar el progreso o el estado de la creación de particiones en un clúster.

    ```kusto
    .show diagnostics
    | project MinPartitioningPercentageInSingleTable,
              TableWithMinPartitioningPercentage
    ```

    The output includes:

    * `MinPartitioningPercentageInSingleTable`: El porcentaje mínimo de datos particionados en todas las tablas que tienen una directiva de particionamiento de datos en el clúster.
      * Si este porcentaje se mantiene constantemente en el 90%, evalúe la capacidad de particionamiento del clúster (consulte [Capacity (capacidad](partitioningpolicy.md#capacity))).
    * `TableWithMinPartitioningPercentage`: El nombre completo de la tabla cuyo porcentaje de particionamiento se muestra arriba.

Use los [comandos. show](commands.md) para supervisar los comandos de creación de particiones y su utilización de recursos. Por ejemplo:

```kusto
.show commands 
| where StartedOn > ago(1d)
| where CommandType == "ExtentsPartition"
| parse Text with ".partition async table " TableName " extents" *
| summarize count(), sum(TotalCpu), avg(tolong(ResourcesUtilization.MemoryPeak)) by TableName, bin(StartedOn, 15m)
| render timechart with(ysplit = panels)
```

#### <a name="capacity"></a>Capacity

* El proceso de creación de particiones de datos tiene como resultado la creación de más extensiones. El clúster puede aumentar gradualmente su [capacidad de combinación de extensiones](../management/capacitypolicy.md#extents-merge-capacity), por lo que el proceso de [combinación de extensiones](../management/extents-overview.md) puede mantenerse al día.
* Si hay un rendimiento de ingesta alto o un gran número de tablas que tienen definida una directiva de particionamiento, el clúster puede aumentar gradualmente su capacidad de [partición de extensión](../management/capacitypolicy.md#extents-partition-capacity), de modo que [el proceso de extensión de las particiones](#the-data-partitioning-process) puede mantenerse al día.
* Para evitar que se consuman demasiados recursos, se limitan estos aumentos dinámicos. Es posible que sea necesario aumentar gradualmente y linealmente más allá del límite si se usan completamente.
  * Si el aumento de las capacidades provoca un aumento significativo del uso de los recursos del clúster, puede escalar el [clúster verticalmente](../../manage-cluster-vertical-scaling.md) / [out](../../manage-cluster-horizontal-scaling.md), ya sea manualmente o habilitando el escalado automático.

### <a name="outliers-in-partitioned-columns"></a>Valores atípicos en columnas con particiones

* Si una clave de partición hash incluye valores que son mucho más comunes que otros, por ejemplo, una cadena vacía o un valor genérico (como `null` o `N/A` ), o que representan una entidad (como `tenant_id` ) más frecuente en el conjunto de datos, que podría contribuir a la distribución desequilibrada de datos entre los nodos del clúster y reducir el rendimiento de las consultas.
* Si una clave de partición de fecha y hora de intervalo uniforme tiene un porcentaje suficientemente grande de valores que están "lejos" de la mayoría de los valores de la columna, por ejemplo, valores de fecha y hora desde el pasado o el futuro, eso podría aumentar la sobrecarga del proceso de creación de particiones de datos y llevar a muchas extensiones pequeñas que el clúster necesitará realizar un seguimiento.

En ambos casos, debe "corregir" los datos o filtrar los registros irrelevantes de los datos antes o en el momento de la ingesta para reducir la sobrecarga de la creación de particiones de datos en el clúster. Por ejemplo, use una [Directiva de actualización](updatepolicy.md)).

## <a name="next-steps"></a>Pasos siguientes

Use los [comandos de control de directivas de partición](../management/partitioning-policy.md) para administrar las directivas de particionamiento de datos para las tablas.

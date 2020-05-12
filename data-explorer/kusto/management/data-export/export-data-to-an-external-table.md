---
title: 'Exportación de datos a una tabla externa: Azure Explorador de datos'
description: En este artículo se describe la exportación de datos a una tabla externa en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 7b4bade1ca874157ec843103a8bcf5236b49abfe
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227798"
---
# <a name="export-data-to-an-external-table"></a>Exportar datos a una tabla externa

Puede exportar datos definiendo una [tabla externa](../externaltables.md) y exportando los datos a ella.
Las propiedades de la tabla se especifican al [crear la tabla externa](../external-tables-azurestorage-azuredatalake.md#create-or-alter-external-table). No es necesario incrustar las propiedades de la tabla en el comando exportar. El comando export hace referencia a la tabla externa por nombre. Exportar datos requiere [permisos de administrador de base](../access-control/role-based-authorization.md)de datos.

**Sintaxis:**

`.export`[ `async` ] `to` `table` *ExternalTableName* <br>
[ `with` `(` *PropertyName* `=` *PropertyValue* `,` ... `)` ] <| *Consulta* de

**Salida:**

|Parámetro de salida |Tipo |Descripción
|---|---|---
|ExternalTableName  |String |Nombre de la tabla externa.
|Ruta de acceso|String|Ruta de acceso de salida.
|NumRecords|String| Número de registros exportados a la ruta de acceso.

**Notas:**
* El esquema de salida de la consulta de exportación debe coincidir con el esquema de la tabla externa, incluidas todas las columnas definidas por las particiones. Por ejemplo, si la tabla tiene particiones por *fecha y hora*, el esquema de salida de la consulta debe tener una columna de marca de tiempo que coincida con *TimestampColumnName*. Este nombre de columna se define en la definición de partición de tabla externa.

* No es posible invalidar las propiedades de la tabla externa mediante el comando export.
 Por ejemplo, no puede exportar datos en formato parquet a una tabla externa cuyo formato de datos sea CSV.

* Las siguientes propiedades se admiten como parte del comando export. Consulte la sección [exportación a almacenamiento](export-data-to-storage.md) para obtener más información: 
   * `sizeLimit`, `parquetRowGroupSize`, `distributed`.

   * Si la tabla externa tiene particiones, los artefactos exportados se escribirán en sus directorios respectivos, de acuerdo con las definiciones de partición que se muestran en el [ejemplo](#partitioned-external-table-example). 

* El número de archivos escritos por partición depende de la configuración:
   * Si la tabla externa solo incluye particiones DateTime o ninguna partición en absoluto: el número de archivos escritos (para cada partición, si existe) debe estar en torno al número de nodos del clúster (o más, si `sizeLimit` se alcanza). Cuando se distribuye la operación de exportación, todos los nodos del clúster se exportan simultáneamente. Para deshabilitar la distribución, de modo que solo un nodo realiza las escrituras, establezca `distributed` en false. Este proceso creará menos archivos, pero reducirá el rendimiento de la exportación.

   * Si la tabla externa incluye una partición por una columna de cadena, el número de archivos exportados debe ser un único archivo por partición (o más, si `sizeLimit` se alcanza). Todos los nodos continúan participando en la exportación (la operación está distribuida), pero cada partición se asigna a un nodo específico. `distributed`En este caso, si se establece en false, solo se realizará la exportación a un solo nodo, pero el comportamiento seguirá siendo el mismo (se escribe un solo archivo por partición).

## <a name="examples"></a>Ejemplos

### <a name="non-partitioned-external-table-example"></a>Ejemplo de tabla externa sin particiones

ExternalBlob es una tabla externa sin particiones. 
```kusto
.export to table ExternalBlob <| T
```

|ExternalTableName|Ruta de acceso|NumRecords|
|---|---|---|
|ExternalBlob|http://storage1.blob.core.windows.net/externaltable1cont1/1_58017c550b384c0db0fea61a8661333e.csv|10|

### <a name="partitioned-external-table-example"></a>Ejemplo de tabla externa con particiones

PartitionedExternalBlob es una tabla externa, que se define de la siguiente manera: 

```kusto
.create external table PartitionedExternalBlob (Timestamp:datetime, CustomerName:string) 
kind=blob
partition by 
   "CustomerName="CustomerName,
   bin(Timestamp, 1d)
dataformat=csv
( 
   h@'http://storageaccount.blob.core.windows.net/container1;secretKey'
)
```

```kusto
.export to table PartitionedExternalBlob <| T
```

|ExternalTableName|Ruta de acceso|NumRecords|
|---|---|---|
|ExternalBlob|http://storageaccount.blob.core.windows.net/container1/CustomerName=customer1/2019/01/01/fa36f35c-c064-414d-b8e2-e75cf157ec35_1_58017c550b384c0db0fea61a8661333e.csv|10|
|ExternalBlob|http://storageaccount.blob.core.windows.net/container1/CustomerName=customer2/2019/01/01/fa36f35c-c064-414d-b8e2-e75cf157ec35_2_b785beec2c004d93b7cd531208424dc9.csv|10|

Si el comando se ejecuta de forma asincrónica (mediante la `async` palabra clave), el resultado está disponible mediante el comando [Mostrar detalles](../operations.md#show-operation-details) de la operación.

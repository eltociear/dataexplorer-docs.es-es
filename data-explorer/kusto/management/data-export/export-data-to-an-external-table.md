---
title: 'Exportación de datos a una tabla externa: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la exportación de datos a una tabla externa en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 56150c480d0d5ecfd4d428e51f7bdb4b68e36b0c
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617704"
---
# <a name="export-data-to-an-external-table"></a>Exportar datos a una tabla externa

Puede exportar datos definiendo una [tabla externa](../externaltables.md) y exportando los datos a ella.
Las propiedades de la tabla se especifican al [crear la tabla externa](../externaltables.md#create-or-alter-external-table); por lo tanto, no es necesario incrustar las propiedades de la tabla en el comando exportar. El comando export hace referencia a la tabla externa por nombre.
Exportar datos requiere [permisos de administrador de base](../access-control/role-based-authorization.md)de datos.

> [!NOTE] 
> * Actualmente no se admite la exportación `impersonate` a una tabla externa con la cadena de conexión.

**Sintaxis:**

`.export`[`async`] `to` `table` *ExternalTableName* <br>
[`with` `(` *PropertyName* PropertyName `=` *PropertyValue*PropertyValue`,`... `)`] <| *Consulta* de

**Salida:**

|Parámetro de salida |Tipo |Descripción
|---|---|---
|ExternalTableName  |String |Nombre de la tabla externa.
|Path|String|Ruta de acceso de salida.
|NumRecords|String| Número de registros exportados a la ruta de acceso.

**Notas:**
* El esquema de salida de la consulta de exportación debe coincidir con el esquema de la tabla externa, incluidas todas las columnas definidas por las particiones. Por ejemplo, si la tabla tiene particiones por *fecha y hora*, el esquema de salida de la consulta debe incluir una columna TIMESTAMP que coincida con el *TimestampColumnName* definido en la definición de la partición de la tabla externa.

* No es posible invalidar las propiedades de la tabla externa mediante el comando export.
 Por ejemplo, no puede exportar datos en formato parquet a una tabla externa cuyo formato de datos sea CSV.

* Las siguientes propiedades se admiten como parte del comando export. Consulte la sección [exportación a almacenamiento](export-data-to-storage.md) para obtener más información: 
   * `sizeLimit`, `parquetRowGroupSize`, `distributed`.

* Si la tabla externa tiene particiones, los artefactos exportados se escribirán en sus directorios respectivos, de acuerdo con las definiciones de partición que se muestran en el [ejemplo](#partitioned-external-table-example). 

* El número de archivos escritos por partición depende de lo siguiente:
   * Si la tabla externa solo incluye particiones DateTime o ninguna partición en absoluto: el número de archivos escritos (para cada partición, si existe) debe estar en torno al número de nodos del clúster (o más, si `sizeLimit` se alcanza). Esto se debe a que la operación de exportación se distribuye, de modo que todos los nodos del clúster se exportan simultáneamente. 
   Para deshabilitar la distribución, de modo que solo un nodo realiza las escrituras `distributed` , establezca en false. Esto creará menos archivos, pero reducirá el rendimiento de la exportación.

   * Si la tabla externa incluye una partición por una columna de cadena, el número de archivos exportados debe ser un único archivo por partición (o `sizeLimit` más, si se alcanza). Todos los nodos continúan participando en la exportación (la operación está distribuida), pero cada partición se asigna a un nodo específico. En `distributed` este caso, si se establece en false, solo se realizará la exportación a un solo nodo, pero el comportamiento seguirá siendo el mismo (se escribe un solo archivo por partición).

## <a name="examples"></a>Ejemplos

### <a name="non-partitioned-external-table-example"></a>Ejemplo de tabla externa sin particiones

ExternalBlob es una tabla externa sin particiones. 
```kusto
.export to table ExternalBlob <| T
```

|ExternalTableName|Path|NumRecords|
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

|ExternalTableName|Path|NumRecords|
|---|---|---|
|ExternalBlob|http://storageaccount.blob.core.windows.net/container1/CustomerName=customer1/2019/01/01/fa36f35c-c064-414d-b8e2-e75cf157ec35_1_58017c550b384c0db0fea61a8661333e.csv|10|
|ExternalBlob|http://storageaccount.blob.core.windows.net/container1/CustomerName=customer2/2019/01/01/fa36f35c-c064-414d-b8e2-e75cf157ec35_2_b785beec2c004d93b7cd531208424dc9.csv|10|

Si el comando se ejecuta de forma asincrónica (mediante `async` la palabra clave), el resultado está disponible mediante el comando [Mostrar detalles](../operations.md#show-operation-details) de la operación.

---
title: 'Exportar datos a una tabla externa: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe Exportar datos a una tabla externa en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: e26d1ae163b69f3a04c52538c245ea446dc11199
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521648"
---
# <a name="export-data-to-an-external-table"></a>Exportar datos a una tabla externa

Puede exportar datos definiendo una [tabla externa](../externaltables.md) y exportándola.
Las propiedades de la tabla se especifican al [crear la tabla externa,](../externaltables.md#create-or-alter-external-table)por lo tanto, no es necesario incrustar las propiedades de la tabla en el comando de exportación. El comando de exportación hace referencia a la tabla externa por su nombre.
Exportar datos requiere permiso de administrador de base de [datos.](../access-control/role-based-authorization.md)

> [!NOTE] 
> * Actualmente no se `impersonate` admite la exportación a una tabla externa con cadena de conexión.

**Sintaxis:**

`.export`[`async` `to` `table` ] *ExternalTableName* <br>
[`with` `(` *PropertyName* `=` *PropertyValue*`,`... `)`] <*Consulta*

**Salida:**

|Parámetro de salida |Tipo |Descripción
|---|---|---
|ExternalTableName  |String |El nombre de la tabla externa.
|Path|String|Ruta de salida.
|NumRecords|String| Número de registros exportados a la ruta de acceso.

**Notas:**
* El esquema de salida de la consulta de exportación debe coincidir con el esquema de la tabla externa, incluidas todas las columnas definidas por las particiones. Por ejemplo, si la tabla tiene particiones de *DateTime*, el esquema de salida de consulta debe incluir una columna Timestamp que coincida con *TimestampColumnName* definido en la definición de particionamiento de tabla externa.

* No es posible invalidar las propiedades de la tabla externa mediante el comando de exportación.
 Por ejemplo, no puede exportar datos en formato Parquet a una tabla externa cuyo formato de datos sea CSV.

* Las siguientes propiedades se admiten como parte del comando de exportación. Consulte la sección [de exportación a almacenamiento](export-data-to-storage.md) para obtener más información: 
   * `sizeLimit`, `parquetRowGroupSize`, `distributed`.

* Si la tabla externa está particionada, los artefactos exportados se escribirán en sus directorios respectivos, según las definiciones de partición como se ve en el [ejemplo.](#partitioned-external-table-example) 

* El número de archivos escritos por partición depende de lo siguiente:
   * Si la tabla externa solo incluye particiones datetime o ninguna partición, el número de archivos escritos (para cada partición, si `sizeLimit` existe) debe estar alrededor del número de nodos del clúster (o más, si se alcanza). Esto se debe a que la operación de exportación se distribuye, de modo que todos los nodos del clúster se exportan simultáneamente. 
   Para deshabilitar la distribución, de modo que solo `distributed` un nodo realice las escrituras, establecido en false. Esto creará menos archivos, pero disminuirá el rendimiento de exportación.

   * Si la tabla externa incluye una partición por una columna de cadena, el número `sizeLimit` de archivos exportados debe ser un único archivo por partición (o más, si se alcanza). Todos los nodos siguen participando en la exportación (la operación se distribuye), pero cada partición se asigna a un nodo específico. Si `distributed` se establece en false en este caso, solo se realizará un único nodo para realizar la exportación, pero el comportamiento seguirá siendo el mismo (un único archivo escrito por partición).

## <a name="examples"></a>Ejemplos

### <a name="non-partitioned-external-table-example"></a>Ejemplo de tabla externa sin particiones

ExternalBlob es una tabla externa sin particiones. 
```kusto
.export to table ExternalBlob <| T
```

|ExternalTableName|Path|NumRecords|
|---|---|---|
|ExternalBlob|http://storage1.blob.core.windows.net/externaltable1cont1/1_58017c550b384c0db0fea61a8661333e.csv|10|

### <a name="partitioned-external-table-example"></a>Ejemplo de tabla externa particionada

PartitionedExternalBlob es una tabla externa, definida de la siguiente manera: 

```
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

```
.export to table PartitionedExternalBlob <| T
```

|ExternalTableName|Path|NumRecords|
|---|---|---|
|ExternalBlob|http://storageaccount.blob.core.windows.net/container1/CustomerName=customer1/2019/01/01/fa36f35c-c064-414d-b8e2-e75cf157ec35_1_58017c550b384c0db0fea61a8661333e.csv|10|
|ExternalBlob|http://storageaccount.blob.core.windows.net/container1/CustomerName=customer2/2019/01/01/fa36f35c-c064-414d-b8e2-e75cf157ec35_2_b785beec2c004d93b7cd531208424dc9.csv|10|

Si el comando se ejecuta de `async` forma asincrónica (mediante la palabra clave), la salida está disponible mediante el comando [show operation details.](../operations.md#show-operation-details)
---
title: 'Administración de directivas de ingesta de streaming de Kusto: Azure Explorador de datos'
description: En este artículo se describe la administración de directivas de ingesta de streaming en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 1c3ce0c0d383d07375333b08de336503d1578b1a
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382002"
---
# <a name="streaming-ingestion-policy-management"></a>Administración de directivas de ingesta de streaming

La Directiva de ingesta de streaming se puede adjuntar a una base de datos o tabla para permitir la ingesta de streaming en esas ubicaciones. La Directiva también define los almacenes de filas usados para la ingesta de streaming.

Para obtener más información sobre la ingesta de streaming [, consulte ingesta de streaming (versión preliminar)](../../ingest-data-streaming.md). Para obtener más información acerca de la Directiva de ingesta de streaming, consulte [Directiva de ingesta de streaming](streamingingestionpolicy.md).

## <a name="show-policy-streamingingestion"></a>. Mostrar streamingingestion de Directiva

El `.show policy streamingingestion` comando muestra la Directiva de ingesta de streaming de la base de datos o de la tabla.

**Sintaxis**

`.show``database`Base de `policy` `streamingingestion` 
 datos `.show` `table`MyTable `policy``streamingingestion`

**Devuelve**

Este comando devuelve una tabla con las columnas siguientes:

|Columna    |Tipo    |Descripción
|---|---|---
|PolicyName|`string`|El nombre de la Directiva: StreamingIngestionPolicy
|EntityName|`string`|Nombre de la base de datos o tabla
|Directiva    |`string`|Un objeto JSON que define la Directiva de ingesta de streaming, con el formato de [objeto de directiva de ingesta de streaming](#streaming-ingestion-policy-object) .

**Ejemplo**

```kusto
.show database DB1 policy streamingingestion 
.show table T1 policy streamingingestion 
```

|PolicyName|EntityName|Directiva|ChildEntities|EntityType|
|---|---|---|---|---|
|StreamingIngestionPolicy|DB1|{"NumberOfRowStores": 4}

### <a name="streaming-ingestion-policy-object"></a>Objeto de directiva de ingesta de streaming

|Propiedad.  |Tipo    |Descripción                                                       |
|----------|--------|------------------------------------------------------------------|
|NumberOfRowStores |`int`  |El número de almacenes de filas asignados a la entidad|
|SealIntervalLimit|`TimeSpan?`|Límite opcional para los intervalos entre las operaciones de sellado en la tabla. El intervalo válido está comprendido entre 1 y 24 horas. Valor predeterminado: 24 horas.|
|SealThresholdBytes|`int?`|Límite opcional para la cantidad de datos que se va a realizar para una sola operación de sellado en la tabla. El intervalo válido para el valor es de entre 10 y 200 MB. Valor predeterminado: 200 MB.|

## <a name="alter-policy-streamingingestion"></a>. Alter Policy streamingingestion

El `.alter policy streamingingestion` comando establece la Directiva streamingingestion de la base de datos o de la tabla.

**Sintaxis**

* `.alter``database`Base de datos `policy` `streamingingestion` *StreamingIngestionPolicyObject*

* `.alter``database`Base de datos `policy` `streamingingestion``enable`

* `.alter``database`Base de datos `policy` `streamingingestion``disable`

* `.alter``table` `policy` MyTable `streamingingestion` *StreamingIngestionPolicyObject*

* `.alter``table` `policy` MyTable `streamingingestion``enable`

* `.alter``table` `policy` MyTable `streamingingestion``disable`

**Notas**

1. *StreamingIngestionPolicyObject* es un objeto JSON que tiene definido el objeto de directiva de ingesta de transmisión por secuencias.

2. `enable`: establezca la Directiva de ingesta de streaming en 4 rowstores si la Directiva no está definida o ya se definió con 0 rowstores, de lo contrario, el comando no hará nada.

3. `disable`: establezca la Directiva de ingesta de streaming en 0 rowstores si la directiva ya se ha definido con rowstores positivos; de lo contrario, el comando no hará nada.

**Ejemplo**

```kusto
.alter database MyDatabase policy streamingingestion '{  "NumberOfRowStores": 4}'

.alter table MyTable policy streamingingestion '{  "NumberOfRowStores": 4}'
```

## <a name="delete-policy-streamingingestion"></a>. eliminar Directiva streamingingestion

El `.delete policy streamingingestion` comando elimina la Directiva streamingingestion de la base de datos o de la tabla.

**Sintaxis** 

`.delete``database`Base de datos `policy``streamingingestion`

`.delete``table`MyTable `policy``streamingingestion`

**Devuelve**

El comando elimina el objeto de directiva streamingingestion de la tabla o la base de datos y, a continuación, devuelve el resultado del comando [. Show Policy streamingingestion](#show-policy-streamingingestion) correspondiente.

**Ejemplo**

```kusto
.delete database MyDatabase policy streamingingestion 
```
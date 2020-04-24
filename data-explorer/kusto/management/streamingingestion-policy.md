---
title: 'Administración de directivas de ingesta de streaming: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la administración de directivas de ingesta de streaming en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: b0b1a76e52688dcc88ca87023309f9c970b4c702
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519625"
---
# <a name="streaming-ingestion-policy-management"></a>Gestión de políticas de ingesta de streaming

La directiva de ingesta de streaming se puede adjuntar a una base de datos o tabla para permitir la ingesta de streaming a esas ubicaciones. La directiva también define los almacenes de filas utilizados para la ingesta de streaming.

Para obtener más información sobre la ingesta de streaming, consulte Ingesta de [streaming (versión preliminar).](https://docs.microsoft.com/azure/data-explorer/ingest-data-streaming) Para obtener más información sobre la directiva de ingesta de streaming, consulte Directiva de [ingesta](streamingingestionpolicy.md)de streaming .

## <a name="show-policy-streamingingestion"></a>.show política streamingingestion

El `.show policy streamingingestion` comando muestra la directiva de ingesta de streaming de la base de datos o la tabla.

**Sintaxis**

`.show``database` `policy` `streamingingestion` MyDatabase 
 `.show` `policy` `table` MyTable`streamingingestion`

**Devuelve**

Este comando devuelve una tabla con las siguientes columnas:

|Columna    |Tipo    |Descripción
|---|---|---
|PolicyName|`string`|El nombre de la directiva - StreamingIngestionPolicy
|EntityName|`string`|Nombre de la base de datos o de la tabla
|Directiva    |`string`|Objeto JSON que define la política de ingesta de streaming, formateada como objeto de política de [ingesta](#streaming-ingestion-policy-object) de streaming

**Ejemplo**

```kusto
.show database DB1 policy streamingingestion 
.show table T1 policy streamingingestion 
```

|PolicyName|EntityName|Directiva|ChildEntities|EntityType|
|---|---|---|---|---|
|StreamingIngestionPolicy|DB1|• "NumberOfRowStores": 4 ?

### <a name="streaming-ingestion-policy-object"></a>Objeto de directiva de ingesta de streaming

|Propiedad  |Tipo    |Descripción                                                       |
|----------|--------|------------------------------------------------------------------|
|NumberOfRowStores |`int`  |El número de almacenes de filas asignados a la entidad|
|SealIntervalLimit|`TimeSpan?`|Límite opcional para los intervalos entre las operaciones de sellado en la tabla. El rango válido es de 1 a 24 horas. Valor predeterminado: 24 horas.|
|SealThresholdBytes|`int?`|Límite opcional para la cantidad de datos que se debe tomar para una sola operación de sellado en la tabla. El rango válido para el valor es entre 10 y 200 MBs. Predeterminado: 200 MBs.|

## <a name="alter-policy-streamingingestion"></a>.alter política streamingingestion

El `.alter policy streamingingestion` comando establece la directiva streamingingestion de la base de datos o tabla.

**Sintaxis**

* `.alter``database` MyDatabase `policy` `streamingingestion` *StreamingIngestionPolicyObject*

* `.alter``database` MyDatabase `policy` `streamingingestion``enable`

* `.alter``database` MyDatabase `policy` `streamingingestion``disable`

* `.alter``table` MyTable `policy` `streamingingestion` *StreamingIngestionPolicyObject*

* `.alter``table` MyTable `policy` `streamingingestion``enable`

* `.alter``table` MyTable `policy` `streamingingestion``disable`

**Notas**

1. *StreamingIngestionPolicyObject* es un objeto JSON que tiene definido el objeto de política de ingesta de streaming.

2. `enable`- establecer la directiva de ingesta de streaming para que esté con 4 almacenes de filas si la directiva no está definida o ya definida con 0 almacenes de filas, de lo contrario el comando no hará nada.

3. `disable`- establecer la directiva de ingesta de streaming para que esté con 0 almacenes de filas si la directiva ya está definida con almacenes de filas positivos, de lo contrario el comando no hará nada.

**Ejemplo**

```kusto
.alter database MyDatabase policy streamingingestion '{  "NumberOfRowStores": 4}'

.alter table MyTable policy streamingingestion '{  "NumberOfRowStores": 4}'
```

## <a name="delete-policy-streamingingestion"></a>.delete política streamingingestion

El `.delete policy streamingingestion` comando elimina la directiva streamingingestion de la base de datos o tabla.

**Sintaxis** 

`.delete``database` MyDatabase `policy``streamingingestion`

`.delete``table` MyTable `policy``streamingingestion`

**Devuelve**

El comando elimina el objeto de directiva streamingingestion de tabla o base de datos y, a continuación, devuelve la salida del comando [.show policy streamingingestion](#show-policy-streamingingestion) correspondiente.

**Ejemplo**

```kusto
.delete database MyDatabase policy streamingingestion 
```
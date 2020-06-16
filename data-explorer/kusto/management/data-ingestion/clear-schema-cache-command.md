---
title: 'Borrado del esquema en caché para la ingesta de streaming: Azure Explorador de datos'
description: En este artículo se describe el comando de administración para borrar esquemas de base de datos en caché en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/20/2020
ms.openlocfilehash: 23d86e528dfe687b79ce225b16712184a9bfcde4
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84784504"
---
# <a name="clear-schema-cache-for-streaming-ingestion"></a>Borrar la caché de esquema para la ingesta de streaming

Los nodos de clúster almacenan en caché el esquema de las bases de datos que reciben datos a través de la ingesta de streaming. Este proceso optimiza el rendimiento y la utilización de los recursos del clúster, pero puede causar retrasos en la propagación cuando cambia el esquema.
Borre la memoria caché para garantizar que las solicitudes de ingesta de streaming posteriores incorporan cambios de base de datos o de esquema de tabla.
Para obtener más información, consulte [ingesta de streaming y cambios de esquema](streaming-ingestion-schema-changes.md).

**Borrar caché de esquema**

El `.clear cache streamingingestion schema` comando vacía el esquema almacenado en memoria caché de todos los nodos del clúster.

**Sintaxis**

`.clear``table` &lt; &gt; nombre `cache` de `streamingingestion` tabla`schema`

`.clear` `database` `cache` `streamingingestion` `schema`

**Devuelve**

Este comando devuelve una tabla con las columnas siguientes:

|Columna    |Tipo    |Descripción
|---|---|---
|NodeId|`string`|Identificador del nodo de clúster
|Estado|`string`|Correcto/error

**Ejemplo**

```kusto
.clear database cache streamingingestion schema

.show table T1 cache streamingingestion schema
```

|NodeId|Estado|
|---|---|
|Nodo1|Correcto
|Nodo2|Con error

> [!NOTE]
> Si se produce un error en el comando o una de las filas de la tabla devuelta contiene *status = failed* , el comando puede reintentarse de forma segura.

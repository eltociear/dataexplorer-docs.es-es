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
ms.openlocfilehash: 55ed390a1c98a307d2bb38476458f29fc9c92997
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84258018"
---
# <a name="streaming-ingestion-policy-management"></a>Administración de directivas de ingesta de streaming

Se puede establecer la Directiva de ingesta de streaming en una tabla para permitir la ingesta de streaming en esta tabla. La Directiva también se puede establecer en el nivel de base de datos para aplicar la misma configuración a las tablas actuales y futuras.

Para obtener más información, consulte [ingesta de streaming](../../ingest-data-streaming.md). Para obtener más información acerca de la Directiva de ingesta de streaming, consulte [Directiva de ingesta de streaming](streamingingestionpolicy.md).

## <a name="display-the-policy"></a>Mostrar la Directiva

El `.show policy streamingingestion` comando muestra la Directiva de ingesta de streaming de la base de datos o de la tabla.
 
**Sintaxis**

`.show``{database|table}` &lt; nombre &gt; de `policy` entidad`streamingingestion`

**Devuelve**

Este comando devuelve una tabla con las columnas siguientes:

|Columna    |Tipo    |Descripción
|---|---|---
|PolicyName|`string`|El nombre de la Directiva: StreamingIngestionPolicy
|EntityName|`string`|Nombre de la base de datos o tabla
|Directiva    |`string`|[objeto de directiva de ingesta de streaming](#streaming-ingestion-policy-object)

**Ejemplos**

```kusto
.show database DB1 policy streamingingestion

.show table T1 policy streamingingestion
```

|PolicyName|EntityName|Directiva|ChildEntities|EntityType|
|---|---|---|---|---|
|StreamingIngestionPolicy|DB1|{"IsEnabled": true, "HintAllocatedRate": null}

## <a name="change-the-policy"></a>Cambiar la directiva

La `.alter[-merge] policy streamingingestion` familia de comandos proporciona medios para modificar la Directiva de ingesta de streaming de la base de datos o de la tabla.

**Sintaxis**

* `.alter``{database|table}` &lt; &gt; nombre `policy` de `streamingingestion` entidad`[enable|disable]`

* `.alter``{database|table}` &lt; objeto de &gt; `policy` `streamingingestion` &lt; [Directiva de ingesta de streaming](#streaming-ingestion-policy-object) de nombre de entidad&gt;

* `.alter-merge``{database|table}` &lt; objeto de &gt; `policy` `streamingingestion` &lt; [Directiva de ingesta de streaming](#streaming-ingestion-policy-object) de nombre de entidad&gt;

> [!Note]
>
> * Permite cambiar el estado habilitado o deshabilitado de la ingesta de streaming sin modificar otras propiedades de la Directiva o establecer las propiedades en valores predeterminados si la Directiva no se definió previamente en la entidad.
>
> * Permite reemplazar toda la Directiva de ingesta de streaming en la entidad. el [objeto de directiva de ingesta de transmisión por secuencias](#streaming-ingestion-policy-object) debe incluir todas las propiedades obligatorias.
>
> * Permite reemplazar solo las propiedades especificadas de la Directiva de ingesta de transmisión por secuencias en la entidad. El [objeto de directiva de ingesta de streaming](#streaming-ingestion-policy-object) puede incluir algunas o ninguna de las propiedades obligatorias.

**Devuelve**

El comando modifica el objeto de directiva de base de datos o `streamingingestion` de tabla y, a continuación, [ `.show policy` `streamingingestion` ](#display-the-policy) devuelve el resultado del comando correspondiente.

**Ejemplos**

```kusto
.alter database DB1 policy streamingingestion enable

.alter table T1 policy streamingingestion disable

.alter database DB1 policy streamingingestion '{"IsEnabled": true, "HintAllocatedRate": 2.1}'

.alter table T1 streamingingestion '{"IsEnabled": true}'

.alter-merge database DB1 policy streamingingestion '{"IsEnabled": false}'

.alter-merge table T1 policy streamingingestion '{"HintAllocatedRate": 1.5}'
```

## <a name="delete-the-policy"></a>Eliminar la Directiva

El `.delete policy streamingingestion` comando elimina la Directiva streamingingestion de la base de datos o de la tabla.

**Sintaxis**

`.delete``{database|table}` &lt; nombre &gt; de `policy` entidad`streamingingestion`

**Devuelve**

El comando elimina el objeto de directiva streamingingestion de la tabla o la base de datos y, a continuación, devuelve el resultado del comando [. Show Policy streamingingestion](#display-the-policy) correspondiente.

**Ejemplos**

```kusto
.delete database DB1 policy streamingingestion

.delete table T1 policy streamingingestion
```

### <a name="streaming-ingestion-policy-object"></a>Objeto de directiva de ingesta de streaming

En la entrada y la salida de los comandos de administración, el objeto de directiva de ingesta de streaming es una cadena con formato JSON que incluye las siguientes propiedades.
|Propiedad.  |Tipo    |Descripción                                                       |Obligatorio/opcional |
|----------|--------|------------------------------------------------------------------|-------|
|IsEnabled |`bool`  |La ingesta de streaming está habilitada para la entidad| Requerido|
|HintAllocatedRate|`double`|Tasa estimada de entrada de datos en GB/hora| Opcionales|

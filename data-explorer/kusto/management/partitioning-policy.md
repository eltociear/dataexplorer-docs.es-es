---
title: 'Administración de directivas de partición de datos: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la administración de directivas de partición de datos en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/04/2020
ms.openlocfilehash: 0e1ff783195f26adf7f98e511ca155f43609098c
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663963"
---
# <a name="data-partitioning-policy-management"></a>Gestión de políticas de partición de datos

La directiva de particionamiento de datos se detalla [aquí](../management/partitioningpolicy.md).

## <a name="show-policy"></a>mostrar política

```kusto
.show table [table_name] policy partitioning
```

El `.show` comando muestra la directiva de particionamiento que se aplica en la tabla.

### <a name="output"></a>Output

|Nombre de la directiva | Nombre de entidad | Directiva | Entidades secundarias | Tipo de entidad
|---|---|---|---|---
|DataPartitioning | Nombre de la tabla | Serialización JSON del objeto de política | null | Tabla

## <a name="alter-and-alter-merge-policy"></a>política de alterar y alterar la fusión

```kusto
.alter table [table_name] policy partitioning @'policy object, serialized as JSON'

.alter-merge table [table_name] policy partitioning @'partial policy object, serialized as JSON'
```

El `.alter` comando permite cambiar la directiva de particionamiento que se aplica en la tabla.

El comando requiere permisos [DatabaseAdmin.](access-control/role-based-authorization.md)

Los cambios en la política podrían tardar hasta 1 hora en surtir efecto.

### <a name="examples"></a>Ejemplos

#### <a name="setting-all-properties-of-the-policy-explicitly-at-table-level"></a>Establecer todas las propiedades de la directiva explícitamente en el nivel de tabla

```kusto
.alter table [table_name] policy partitioning @'{'
  '"PartitionKeys": ['
    '{'
      '"ColumnName": "my_string_column",'
      '"Kind": "Hash",'
      '"Properties": {'
        '"Function": "XxHash64",'
        '"MaxPartitionCount": 256,'
      '}'
    '},'
    '{'
      '"ColumnName": "my_datetime_column",'
      '"Kind": "UniformRange",'
      '"Properties": {'
        '"Reference": "1970-01-01T00:00:00",'
        '"RangeSize": "1.00:00:00"'
      '}'
    '}'
  ']'
'}'
```

#### <a name="setting-a-specific-property-of-the-policy-explicitly-at-table-level"></a>Establecer una propiedad específica de la directiva explícitamente a nivel de tabla

Para establecer `EffectiveDateTime` la directiva en un valor diferente, utilice el siguiente comando:

```kusto
.alter table [table_name] policy partitioning @'{"EffectiveDateTime":"2020-01-01"}'
```

## <a name="delete-policy"></a>política de eliminación

```kusto
.delete table [table_name] policy partitioning
```

El `.delete` comando elimina la directiva de particionamiento de la tabla dada.

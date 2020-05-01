---
title: 'Administración de directivas de particionamiento de datos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la administración de directivas de particionamiento de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/04/2020
ms.openlocfilehash: 1ad9b359422b51084f1be1c64d27d656313d9296
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616327"
---
# <a name="data-partitioning-policy-management"></a>Administración de directivas de particionamiento de datos

La Directiva de creación de particiones de datos se detalla [aquí](../management/partitioningpolicy.md).

## <a name="show-policy"></a>Mostrar Directiva

```kusto
.show table [table_name] policy partitioning
```

El `.show` comando muestra la Directiva de particionamiento que se aplica a la tabla.

### <a name="output"></a>Output

|Nombre de la directiva | Nombre de entidad | Directiva | Entidades secundarias | Tipo de entidad
|---|---|---|---|---
|Particionar | Nombre de la tabla | Serialización de JSON del objeto de Directiva | null | Tabla

## <a name="alter-and-alter-merge-policy"></a>modificar y modificar la Directiva de combinación

```kusto
.alter table [table_name] policy partitioning @'policy object, serialized as JSON'

.alter-merge table [table_name] policy partitioning @'partial policy object, serialized as JSON'
```

El `.alter` comando permite cambiar la Directiva de particionamiento que se aplica en la tabla.

El comando requiere permisos [DatabaseAdmin](access-control/role-based-authorization.md) .

Los cambios en la Directiva pueden tardar hasta 1 hora en surtir efecto.

### <a name="examples"></a>Ejemplos

#### <a name="setting-all-properties-of-the-policy-explicitly-at-table-level"></a>Establecer todas las propiedades de la Directiva explícitamente en el nivel de tabla

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

#### <a name="setting-a-specific-property-of-the-policy-explicitly-at-table-level"></a>Establecer una propiedad específica de la Directiva explícitamente en el nivel de tabla

Para establecer la `EffectiveDateTime` de la Directiva en un valor diferente, use el siguiente comando:

```kusto
.alter-merge table [table_name] policy partitioning @'{"EffectiveDateTime":"2020-01-01"}'
```

## <a name="delete-policy"></a>eliminar Directiva

```kusto
.delete table [table_name] policy partitioning
```

El `.delete` comando elimina la Directiva de particionamiento de la tabla especificada.

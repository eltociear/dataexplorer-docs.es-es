---
title: 'Combinar la administración de directivas: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la administración de directivas de mezcla en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4093c9c09e4e268bc38cabdc6da27f0ac2ee17ab
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81664019"
---
# <a name="merge-policy-management"></a>Combinar la gestión de políticas

## <a name="show-policy"></a>mostrar política

```kusto
.show table [table_name] policy merge

.show table * policy merge

.show database [database_name] policy merge

.show database * policy merge
```

Muestra la directiva de combinación actual para la base de datos o la tabla.
Muestra todas las directivas del tipo de entidad especificado (base de datos o tabla) si el nombre especificado es '*'.

### <a name="output"></a>Output

|Nombre de la directiva | Nombre de entidad | Directiva | Entidades secundarias | Tipo de entidad
|---|---|---|---|---
|ExtentsMergePolicy | Base de datos / Nombre de la tabla | una cadena con formato JSON que representa la política | Una lista de tablas (para una base de datos)|Tabla de &#124; de base de datos

## <a name="alter-policy"></a>alterar la política

### <a name="examples"></a>Ejemplos

#### <a name="1-setting-all-properties-of-the-policy-explicitly-at-table-level"></a>1. Establecer todas las propiedades de la política explícitamente, a nivel de tabla:

```kusto
.alter table [table_name] policy merge 
@'{'
    '"ExtentSizeTargetInMb": 1024,'
    '"OriginalSizeInMbUpperBoundForRebuild": 2048,'
    '"RowCountUpperBoundForRebuild": 750000, '
    '"RowCountUpperBoundForMerge": 0, '
    '"MaxExtentsToMerge": 100, '
    '"LoopPeriod": "01:00:00", '
    '"MaxRangeInHours": 8, '
    '"AllowRebuild": true,'
    '"AllowMerge": true '
'}'
```

#### <a name="2-setting-all-properties-of-the-policy-explicitly-at-database-level"></a>2. Establecer todas las propiedades de la directiva explícitamente, a nivel de base de datos:

```kusto
.alter database [database_name] policy merge 
@'{'
    '"ExtentSizeTargetInMb": 1024,'
    '"OriginalSizeInMbUpperBoundForRebuild": 2048,'
    '"RowCountUpperBoundForRebuild": 750000,'
    '"RowCountUpperBoundForMerge": 0,'
    '"MaxExtentsToMerge": 100,'
    '"LoopPeriod": "01:00:00",'
    '"MaxRangeInHours": 8,'
    '"AllowRebuild": true,'
    '"AllowMerge": true'
'}'
```

#### <a name="3-setting-the-default-merge-policy-at-database-level"></a>3. Establecer la directiva de combinación *predeterminada* en el nivel de base de datos:

```kusto
.alter database [database_name] policy merge '{}'
```

#### <a name="4-altering-a-single-property-of-the-policy-at-database-level-keeping-all-other-properties-as-is"></a>4. Alterar una sola propiedad de la directiva a nivel de base de datos, manteniendo todas las demás propiedades tal como está:

```kusto
.alter-merge database [database_name] policy merge
@'{'
    '"ExtentSizeTargetInMb": 1024'
'}'
```

#### <a name="5-altering-a-single-property-of-the-policy-at-table-level-keeping-all-other-properties-as-is"></a>5. Alterar una sola propiedad de la política a nivel de tabla, manteniendo todas las demás propiedades tal como está:

```kusto
.alter-merge table [table_name] policy merge
@'{'
    '"RowCountUpperBoundForRebuild": 750000'
'}'
```

Todo lo anterior devuelve la directiva de combinación de extensiones actualizadas para la entidad (base de datos o tabla especificada como un nombre completo) como salida.

Los cambios en la política podrían tardar hasta 1 hora en surtir efecto.

## <a name="delete-policy-of-merge"></a>política de eliminación de la fusión

```kusto
.delete table [table_name] policy merge

.delete database [database_name] policy merge

```

El comando elimina la directiva de combinación actual para la entidad determinada.

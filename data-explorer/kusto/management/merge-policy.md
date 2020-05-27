---
title: 'Administración de directivas de mezcla: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la administración de directivas de combinación en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9ef6f2cd2359e35e90c3903738adf82728e9da40
ms.sourcegitcommit: a562ce255ac706ca1ca77d272a97b5975235729d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83867076"
---
# <a name="merge-policy-management"></a>Administración de directivas de mezcla

## <a name="show-policy"></a>Mostrar Directiva

```kusto
.show table [table_name] policy merge

.show table * policy merge

.show database [database_name] policy merge

.show database * policy merge
```

Muestra la Directiva de mezcla actual para la base de datos o la tabla.
Muestra todas las directivas del tipo de entidad dado (base de datos o tabla) si el nombre especificado es ' * '.

### <a name="output"></a>Output

|Nombre de la directiva | Nombre de entidad | Directiva | Entidades secundarias | Tipo de entidad
|---|---|---|---|---
|ExtentsMergePolicy | Nombre de la base de datos o tabla | una cadena con formato JSON que representa la Directiva. | Una lista de tablas (para una base de datos)|Tabla de &#124; de base de datos

## <a name="alter-policy"></a>modificar Directiva

### <a name="examples"></a>Ejemplos

#### <a name="1-setting-all-properties-of-the-policy-explicitly-at-table-level"></a>1. establecer todas las propiedades de la Directiva explícitamente, en el nivel de tabla:

```kusto
.alter table [table_name] policy merge 
@'{'
    '"ExtentSizeTargetInMb": 1024,'
    '"OriginalSizeInMbUpperBoundForRebuild": 2048,'
    '"OriginalSizeInMbUpperBoundForMerge": 4096,'
    '"RowCountUpperBoundForRebuild": 750000,'
    '"RowCountUpperBoundForMerge": 0,'
    '"MaxExtentsToMerge": 100,'
    '"LoopPeriod": "01:00:00",'
    '"MaxRangeInHours": 8,'
    '"AllowRebuild": true,'
    '"AllowMerge": true '
'}'
```

#### <a name="2-setting-all-properties-of-the-policy-explicitly-at-database-level"></a>2. establecer todas las propiedades de la Directiva explícitamente, en el nivel de base de datos:

```kusto
.alter database [database_name] policy merge 
@'{'
    '"ExtentSizeTargetInMb": 1024,'
    '"OriginalSizeInMbUpperBoundForRebuild": 2048,'
    '"OriginalSizeInMbUpperBoundForMerge": 4096,'
    '"RowCountUpperBoundForRebuild": 750000,'
    '"RowCountUpperBoundForMerge": 0,'
    '"MaxExtentsToMerge": 100,'
    '"LoopPeriod": "01:00:00",'
    '"MaxRangeInHours": 8,'
    '"AllowRebuild": true,'
    '"AllowMerge": true'
'}'
```

#### <a name="3-setting-the-default-merge-policy-at-database-level"></a>3. establecer la Directiva de combinación *predeterminada* en el nivel de base de datos:

```kusto
.alter database [database_name] policy merge '{}'
```

#### <a name="4-altering-a-single-property-of-the-policy-at-database-level-keeping-all-other-properties-as-is"></a>4. modificar una única propiedad de la Directiva en el nivel de base de datos, manteniendo todas las demás propiedades tal cual:

```kusto
.alter-merge database [database_name] policy merge
@'{'
    '"ExtentSizeTargetInMb": 1024'
'}'
```

#### <a name="5-altering-a-single-property-of-the-policy-at-table-level-keeping-all-other-properties-as-is"></a>5. modificar una única propiedad de la Directiva en el nivel de tabla, manteniendo todas las demás propiedades tal cual:

```kusto
.alter-merge table [table_name] policy merge
@'{'
    '"RowCountUpperBoundForRebuild": 750000'
'}'
```

Todo lo anterior devuelve la Directiva de combinación de extensiones actualizadas para la entidad (base de datos o tabla especificada como un nombre completo) como salida.

Los cambios en la Directiva pueden tardar hasta 1 hora en surtir efecto.

## <a name="delete-policy-of-merge"></a>eliminar Directiva de combinación

```kusto
.delete table [table_name] policy merge

.delete database [database_name] policy merge

```

El comando elimina la Directiva de mezcla actual para la entidad especificada.

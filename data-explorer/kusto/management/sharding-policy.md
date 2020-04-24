---
title: 'Administración de directivas de particiones: Explorador de azure Data Explorer ( Sharding Policy Management ) Microsoft Docs'
description: En este artículo se describe la administración de directivas de partición en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7770e0834e4b00f42158732e667d41eb636cec5b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520050"
---
# <a name="sharding-policy-management"></a>Gestión de políticas de partición

## <a name="show-policy"></a>mostrar política

```kusto
.show table [table_name] policy sharding

.show table * policy sharding

.show database [database_name] policy sharding
```

`Show`directiva muestra la directiva de partición para la base de datos o tabla. Muestra todas las directivas del tipo de entidad especificado (base de datos o tabla) si el nombre especificado es '*'.

### <a name="output"></a>Output

|Nombre de la directiva | Nombre de entidad | Directiva | Entidades secundarias | Tipo de entidad
|---|---|---|---|---
|ExtentsShardingPolicy | base de datos / nombre de la tabla | cadena de formato json que representa la política | lista de tablas (para una base de datos)|base de datos / tabla

## <a name="alter-policy"></a>alterar la política

### <a name="examples"></a>Ejemplos

Los siguientes ejemplos devuelven la directiva de partición de extensiones actualizada para la entidad, con la base de datos o la tabla especificada como un nombre completo, como salida.

#### <a name="setting-all-properties-of-the-policy-explicitly-at-table-level"></a>Establecer todas las propiedades de la directiva explícitamente en el nivel de tabla

```kusto
.alter table [table_name] policy sharding 
@'{ "MaxRowCount": 750000, "MaxExtentSizeInMb": 1024, "MaxOriginalSizeInMb": 2048}'
```

#### <a name="setting-all-properties-of-the-policy-explicitly-at-database-level"></a>Establecer todas las propiedades de la directiva explícitamente en el nivel de base de datos

```kusto
.alter database [database_name] policy sharding
@'{ "MaxRowCount": 750000, "MaxExtentSizeInMb": 1024, "MaxOriginalSizeInMb": 2048}'
```

#### <a name="setting-the-default-sharding-policy-at-database-level"></a>Establecer la directiva de partición *predeterminada* a nivel de base de datos

```kusto
.alter database [database_name] policy sharding @'{}'
```

#### <a name="altering-a-single-property-of-the-policy-at-database-level"></a>Modificar una sola propiedad de la directiva a nivel de base de datos 

Mantenga todas las demás propiedades tal como están.

```kusto
.alter-merge database [database_name] policy sharding
@'{ "MaxExtentSizeInMb": 1024}'
```

#### <a name="altering-a-single-property-of-the-policy-at-table-level"></a>Alterar una sola propiedad de la política a nivel de tabla

Mantenga todas las demás propiedades tal como está

```kusto
.alter-merge table [table_name] policy sharding
@'{ "MaxRowCount": 750000}'
```

## <a name="delete-policy"></a>política de eliminación

```kusto
.delete table [table_name] policy sharding

.delete database [database_name] policy sharding
```

El comando elimina la directiva de partición actual para la entidad determinada.
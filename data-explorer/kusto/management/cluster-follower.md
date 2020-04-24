---
title: 'Comandos de seguimiento de clúster: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen los comandos de seguimiento de clúster de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2020
ms.openlocfilehash: 73b65cc59ff98bc658c542d690812ccf5ad3895a
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108463"
---
# <a name="cluster-follower-commands"></a>Comandos de seguimiento de clústeres

A continuación se enumeran los comandos de control para administrar la configuración de clúster del seguimiento. Estos comandos se ejecutan sincrónicamente, pero se aplican en la siguiente actualización periódica del esquema. Por lo tanto, pueden transcurrir unos minutos hasta que se aplique la nueva configuración.

Los comandos de seguimiento incluyen [comandos de nivel de base de datos](#database-level-commands) y [comandos de nivel de tabla](#table-level-commands).

## <a name="database-level-commands"></a>Comandos de nivel de base de datos

### <a name="show-follower-database"></a>. Mostrar la base de datos del seguimiento

Muestra una base de datos (o bases de datos) seguida de otro clúster de líderes, que tiene una o más invalidaciones de nivel de base de datos configuradas.


**Sintaxis**

`.show` `follower` `database` *DatabaseName*

`.show``follower` `,`DatabaseName1... *DatabaseName1* `databases` `(` `,` *DatabaseNameN*`)`

**Salida** 

| Parámetro de salida                     | Tipo    | Descripción                                                                                                        |
|--------------------------------------|---------|--------------------------------------------------------------------------------------------------------------------|
| DatabaseName                         | String  | Nombre de la base de datos que se va a seguir.                                                                           |
| LeaderClusterMetadataPath            | String  | La ruta de acceso al contenedor de metadatos del clúster de la directriz.                                                               |
| CachingPolicyOverride                | String  | Una directiva de almacenamiento en caché invalidada para la base de datos, serializada como JSON o null.                                         |
| AuthorizedPrincipalsOverride         | String  | Una colección de invalidaciones de entidades de seguridad autorizadas para la base de datos, serializada como JSON o null.                    |
| AuthorizedPrincipalsModificationKind | String  | Tipo de modificación que se va a aplicar`none`mediante `union` AuthorizedPrincipalsOverride `replace`(, o).                  |
| CachingPoliciesModificationKind      | String  | Tipo de modificación que se va a aplicar mediante invalidaciones de directiva de almacenamiento en`none`caché `union` de `replace`nivel de tabla o base de datos (o). |
| IsAutoPrefetchEnabled                | Boolean | Si los nuevos datos se capturan previamente en cada actualización del esquema.        |
| TableMetadataOverrides               | String  | Una serialización de JSON de las invalidaciones de propiedad de nivel de tabla (si se han definido).                                      |

### <a name="alter-follower-database-policy-caching"></a>. Alter follower Database Policy Caching

Modifica una directiva de almacenamiento en caché de la base de datos de seguimiento para invalidar el conjunto en la base de datos de origen en el clúster de líderes. Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md).



**Notas**

* El valor `modification kind` predeterminado para las directivas `union`de almacenamiento en caché es. Para cambiar el `modification kind` uso del comando [. Alter follower Database Caching-Policies-modification-Kind](#alter-follower-database-caching-policies-modification-kind) .
* Ver la Directiva o las directivas vigentes después del cambio se puede realizar `.show` mediante los comandos:
    * [. Mostrar retención de directiva de base de datos](../management/retention-policy.md#show-retention-policy)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
    * [.show table details](show-tables-command.md)
* La visualización de la configuración de invalidación en la base de datos de seguimiento una vez realizado el cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.alter``follower` `=` *HotDataSpan* *DatabaseName* NombreDeBaseDeDatos HotDataSpan `policy` `caching` `database` `hot`



**Ejemplo**



```kusto
.alter follower database MyDb policy caching hot = 7d
```

### <a name="delete-follower-database-policy-caching"></a>. eliminar almacenamiento en caché de directiva de base de datos de seguimiento

Elimina una directiva de almacenamiento en caché de invalidación de base de datos de seguimiento. Esto hace que la directiva establecida en la base de datos de origen del clúster líder sea la efectiva.
Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md). 

**Notas**

* Ver la Directiva o las directivas vigentes después del cambio se puede realizar `.show` mediante los comandos:
    * [. Mostrar retención de directiva de base de datos](../management/retention-policy.md#show-retention-policy)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
    * [.show table details](show-tables-command.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.delete` `follower` `database` *DatabaseName* `policy` `caching`

**Ejemplo**

```kusto
.delete follower database MyDB policy caching
```

### <a name="add-follower-database-principals"></a>. agregar entidades de seguridad de base de datos de seguimiento

Agrega entidades de seguridad autorizadas a la colección de base de datos de seguimiento de las entidades de seguridad autorizadas de invalidación. Requiere el [permiso DatabaseAdmin](../management/access-control/role-based-authorization.md).



**Notas**

* El valor `modification kind` predeterminado para estas entidades de seguridad `none`autorizadas es. Para cambiar el `modification kind` uso de las [entidades de seguridad de la base de datos Alter follower-modification-Kind](#alter-follower-database-principals-modification-kind).
* Ver la colección efectiva de entidades de seguridad después de que el cambio se puede `.show` realizar mediante los comandos:
    * [. Mostrar entidades de seguridad de base de datos](../management/security-roles.md#managing-database-security-roles)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.add``admins` | `users` *DatabaseName* `(` *Principal1*de rol databasename`monitors`( | )...`viewers` |  `follower` `database` `,` `,` *principaln* `)` [`'`*notes*notas`'`]




**Ejemplo**

```kusto
.add follower database MyDB viewers ('aadgroup=mygroup@microsoft.com') 'My Group'
```

```kusto

```

### <a name="drop-follower-database-principals"></a>. quitar entidades de seguridad de base de datos de seguimiento

Quita las entidades de seguridad autorizadas de la colección de bases de datos de seguimiento de las entidades de seguridad autorizadas de invalidación.
Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md).

**Notas**

* Ver la colección efectiva de entidades de seguridad después de que el cambio se puede `.show` realizar mediante los comandos:
    * [. Mostrar entidades de seguridad de base de datos](../management/security-roles.md#managing-database-security-roles)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.drop``follower` `monitors` *DatabaseName* `(` *principal1*DatabaseName (`admins` | `users`) principal1`,`. | .. `database` `viewers` |  `,` *entidad* de seguridad`)`

**Ejemplo**

```kusto
.drop follower database MyDB viewers ('aadgroup=mygroup@microsoft.com')
```

### <a name="alter-follower-database-principals-modification-kind"></a>. Alter follower Database entidades-modification-Kind

Modifica el tipo de modificación de entidades de seguridad autorizadas de la base de datos de seguimiento. Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md).

**Notas**

* Ver la colección efectiva de entidades de seguridad después de que el cambio se puede `.show` realizar mediante los comandos:
    * [. Mostrar entidades de seguridad de base de datos](../management/security-roles.md#managing-database-security-roles)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.alter``follower`  |  *DatabaseName*  | Databasename`replace`= (`none`) `database` 
 `principals-modification-kind` `union`



**Ejemplo**

```kusto
.alter follower database MyDB principals-modification-kind = union
```

### <a name="alter-follower-database-caching-policies-modification-kind"></a>. Alter follower Database Caching-Policies-modification-Kind

Modifica el tipo de modificación de directivas de almacenamiento en caché de tabla y base de datos de seguimiento. Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md).

**Notas**

* Ver la colección efectiva de directivas de almacenamiento en la base de datos o de tabla después del cambio se puede `.show` realizar mediante los comandos estándar:
    * [. Mostrar detalles de las tablas](show-tables-command.md)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.alter``follower` `union` *DatabaseName* `replace`DatabaseName = ()`none` |  `database` `caching-policies-modification-kind`  | 



**Ejemplo**

```kusto
.alter follower database MyDB caching-policies-modification-kind = union
```



## <a name="table-level-commands"></a>Comandos de nivel de tabla

### <a name="alter-follower-table-policy-caching"></a>. Alter de la Directiva de tabla de seguimiento del almacenamiento en caché

Modifica una directiva de almacenamiento en caché de nivel de tabla en la base de datos de seguimiento para invalidar la directiva establecida en la base de datos de origen en el clúster de líderes.
Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md). 



**Notas**

* Ver la Directiva o las directivas vigentes después del cambio se puede realizar `.show` mediante los comandos:
    * [. Mostrar retención de directiva de base de datos](../management/retention-policy.md#show-retention-policy)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
    * [.show table details](show-tables-command.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**





`.alter``follower` `=` *HotDataSpan* *DatabaseName* `caching` *TableName* NombreDeBaseDeDatos Table TableName `policy` HotDataSpan `database` `hot`

`.alter``follower` *DatabaseName* `(` *TableName1*NombreDeBaseDeDatos tables TableName1`,`... `database` `,` *TableNameN* TableNameN`)` *HotDataSpan* HotDataSpan `policy` `caching` `hot` `=`

**Ejemplo**



```kusto
.alter follower database MyDb tables (Table1, Table2) policy caching hot = 7d
```

### <a name="delete-follower-table-policy-caching"></a>. eliminar el almacenamiento en caché de directiva de tabla de seguimiento

Elimina una directiva de almacenamiento en caché de nivel de tabla en la base de datos de seguimiento, lo que permite que la directiva establecida en la base de datos de origen del clúster líder sea la efectiva. Requiere [permisos de DatabaseAdmin](../management/access-control/role-based-authorization.md). 

**Notas**

* Ver la Directiva o las directivas vigentes después del cambio se puede realizar `.show` mediante los comandos:
    * [. Mostrar retención de directiva de base de datos](../management/retention-policy.md#show-retention-policy)
    * [. Mostrar detalles de la base de datos](../management/show-databases.md)
    * [.show table details](show-tables-command.md)
* Ver la configuración de invalidación en la base de datos del seguimiento después del cambio se puede realizar mediante [. Mostrar la base de datos del seguimiento](#show-follower-database)

**Sintaxis**

`.delete``follower` `table` *TableName* *DatabaseName* NombreDeBaseDeDatos TableName `database` `policy``caching`

`.delete``follower` `(` *TableName1* *DatabaseName* NombreDeBaseDeDatos `tables` TableName1`,`... `database` `,``)` *TableNameN* TableNameN `policy``caching`

**Ejemplo**

```kusto
.delete follower database MyDB tables (Table1, Table2) policy caching
```

## <a name="sample-configuration"></a>Configuración de ejemplo

A continuación se muestran ejemplos de pasos para configurar una base de datos de seguimiento.

En este ejemplo:

* Nuestro clúster `MyFollowerCluster` de seguimiento será la siguiente base de `MyDatabase` datos del clúster de líderes `MyLeaderCluster`,.
    * `MyDatabase`tiene `N` tablas: `MyTable1`, `MyTable2`, `MyTable3`,... `MyTableN` (`N` > 3).
    * En `MyLeaderCluster`:
    
    | `MyTable1`Directiva de almacenamiento en caché | `MyTable2`Directiva de almacenamiento en caché | `MyTable3`... `MyTableN` Directiva de almacenamiento en caché   | `MyDatabase`Entidades de seguridad autorizadas                                                    |
    |---------------------------|---------------------------|------------------------------------------|---------------------------------------------------------------------------------------|
    | intervalo de datos activos =`7d`      | intervalo de datos activos =`30d`     | intervalo de datos activos =`365d`                   | *Visores* = ;`aadgroup=scubadivers@contoso.com` *Administradores* de = `aaduser=jack@contoso.com` |
     
    * En `MyFollowerCluster` lo que queremos:
    
    | `MyTable1`Directiva de almacenamiento en caché | `MyTable2`Directiva de almacenamiento en caché | `MyTable3`... `MyTableN` Directiva de almacenamiento en caché   | `MyDatabase`Entidades de seguridad autorizadas                                                    |
    |---------------------------|---------------------------|------------------------------------------|---------------------------------------------------------------------------------------|
    | intervalo de datos activos =`1d`      | intervalo de datos activos =`3d`      | intervalo de datos activos `0d` = (no hay nada almacenado en caché) | *Admins*`aaduser=jack@contoso.com` *Viewers* Administradores = , visores = `aaduser=jill@contoso.com`         |

> [!IMPORTANT] 
> `MyFollowerCluster` Y `MyLeaderCluster` deben estar en la misma región.

### <a name="steps-to-execute"></a>Pasos para ejecutar

*Requisito previo:* Configure `MyFollowerCluster` el clúster para `MyDatabase` que siga `MyLeaderCluster`la base de datos del clúster.

> [!NOTE]
> Se espera que la entidad de seguridad que ejecuta los comandos `DatabaseAdmin` de control `MyDatabase`sea una base de datos.

#### <a name="show-the-current-configuration"></a>Mostrar la configuración actual

Vea la configuración actual según la que `MyDatabase` se va a seguir `MyFollowerCluster`:

```kusto
.show follower database MyDatabase
| evaluate narrow() // just for presentation purposes
```

| Columna                              | Value                                                    |
|-------------------------------------|----------------------------------------------------------|
|DatabaseName                         | MyDatabase                                               |
|LeaderClusterMetadataPath            | `https://storageaccountname.blob.core.windows.net/cluster` |
|CachingPolicyOverride                | nulo                                                     |
|AuthorizedPrincipalsOverride         | []                                                       |
|AuthorizedPrincipalsModificationKind | None                                                     |
|IsAutoPrefetchEnabled                | False                                                    |
|TableMetadataOverrides               |                                                          |
|CachingPoliciesModificationKind      | Union                                                    |                                                                                                                      |

#### <a name="override-authorized-principals"></a>Invalidar entidades de seguridad autorizadas

Reemplace la colección de entidades de seguridad `MyDatabase` autorizadas `MyFollowerCluster` de en con una recopilación que incluya solo un usuario de AAD como administrador de la base de datos y un usuario de AAD como visor de bases de datos:

```kusto
.add follower database MyDatabase admins ('aaduser=jack@contoso.com')

.add follower database MyDatabase viewers ('aaduser=jill@contoso.com')

.alter follower database MyDatabase principals-modification-kind = replace
```

Solo las dos entidades de seguridad específicas están autorizadas `MyDatabase` para obtener acceso en`MyFollowerCluster`

```kusto
.show database MyDatabase principals
```

| Rol                       | PrincipalType | PrincipalDisplayName                        | PrincipalObjectId                    | PrincipalFQN                                                                      | Notas |
|----------------------------|---------------|---------------------------------------------|--------------------------------------|-----------------------------------------------------------------------------------|-------|
| Admin. de base de datos  | Usuario de AAD      | Jack Kusto (UPN: jack@contoso.com)       | 12345678-ABCD-EFEF-1234-350bf486087b | aaduser = 87654321-ABCD-EFEF-1234-350bf486087b; 55555555-4444-3333-2222-2d7cd011db47 |       |
| Visor de bases de datos Database | Usuario de AAD      | Jill Kusto (UPN: jack@contoso.com)       | abcdefab-abcd-efef-1234-350bf486087b | aaduser = 54321789-ABCD-EFEF-1234-350bf486087b; 55555555-4444-3333-2222-2d7cd011db47 |       |

```kusto
.show follower database MyDatabase
| mv-expand parse_json(AuthorizedPrincipalsOverride)
| project AuthorizedPrincipalsOverride.Principal.FullyQualifiedName
```

| AuthorizedPrincipalsOverride_Principal_FullyQualifiedName                         |
|-----------------------------------------------------------------------------------|
| aaduser = 87654321-ABCD-EFEF-1234-350bf486087b; 55555555-4444-3333-2222-2d7cd011db47 |
| aaduser = 54321789-ABCD-EFEF-1234-350bf486087b; 55555555-4444-3333-2222-2d7cd011db47 |

#### <a name="override-caching-policies"></a>Invalidar directivas de almacenamiento en caché

Reemplace la colección de directivas de almacenamiento en caché de nivel de `MyDatabase` tabla `MyFollowerCluster` y de base de datos *not* de en; para ello, configure todas las tablas para `MyTable1`que `MyTable2` no se almacenen sus datos en la memoria caché, `1d` excepto `3d`dos tablas específicas, que tendrán sus datos almacenados en caché para los períodos de y, respectivamente:

```kusto
.alter follower database MyDatabase policy caching hot = 0d

.alter follower database MyDatabase table MyTable1 policy caching hot = 1d

.alter follower database MyDatabase table MyTable2 policy caching hot = 3d

.alter follower database MyDatabase caching-policies-modification-kind = replace
```

Solo se almacenan en caché los datos de las dos tablas específicas y el resto de las tablas tiene un período `0d`de datos activo de:

```kusto
.show tables details
| summarize TableNames = make_list(TableName) by CachingPolicy
```
| CachingPolicy                                                                | TableNames                  |
|------------------------------------------------------------------------------|-----------------------------|
| {"DataHotSpan": {"Value": "1.00:00:00"}, "IndexHotSpan": {"Value": "1.00:00:00"}} | ["MyTable1"]                |
| {"DataHotSpan": {"Value": "3,00:00:00"}, "IndexHotSpan": {"Value": "3,00:00:00"}} | ["MyTable2"]                |
| {"DataHotSpan": {"Value": "0,00:00:00"}, "IndexHotSpan": {"Value": "0,00:00:00"}} | ["MyTable3",..., "MyTable"] |

```kusto
.show follower database MyDatabase
| mv-expand parse_json(TableMetadataOverrides)
| project TableMetadataOverrides
```

| TableMetadataOverrides                                                                                              |
|---------------------------------------------------------------------------------------------------------------------|
| {"MyTable1": {"CachingPolicyOverride": {"DataHotSpan": {"Value": "1.00:00:00"}, "IndexHotSpan": {"Value": "1.00:00:00"}}}} |
| {"MyTable2": {"CachingPolicyOverride": {"DataHotSpan": {"Value": "3,00:00:00"}, "IndexHotSpan": {"Value": "3,00:00:00"}}}} |

#### <a name="summary"></a>Resumen

Vea la configuración actual en `MyDatabase` la que se va `MyFollowerCluster`a seguir:

```kusto
.show follower database MyDatabase
| evaluate narrow() // just for presentation purposes
```

| Columna                              | Value                                                                                                                                                                           |
|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|DatabaseName                         | MyDatabase                                                                                                                                                                      |
|LeaderClusterMetadataPath            | `https://storageaccountname.blob.core.windows.net/cluster`                                                                                                                        |
|CachingPolicyOverride                | {"DataHotSpan": {"Value": "00:00:00"}, "IndexHotSpan": {"Value": "00:00:00"}}                                                                                                        |
|AuthorizedPrincipalsOverride         | [{"Principal": {"FullyQualifiedName": "aaduser = 87654321-ABCD-EFEF-1234-350bf486087b",...}, {"entidad de seguridad": {"FullyQualifiedName": "aaduser = 54321789-ABCD-EFEF-1234-350bf486087b",...}] |
|AuthorizedPrincipalsModificationKind | Replace                                                                                                                                                                         |
|IsAutoPrefetchEnabled                | False                                                                                                                                                                           |
|TableMetadataOverrides               | {"MyTargetTable": {"CachingPolicyOverride": {"DataHotSpan": {"Value": "3,00:00:00"}...}, "MySourceTable": {"CachingPolicyOverride": {"DataHotSpan": {"Value": "1.00:00:00"},...}}}       |
|CachingPoliciesModificationKind      | Replace                                                                                                                                                                         |
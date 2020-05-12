---
title: 'Comandos de control de tabla externos: Azure Explorador de datos'
description: En este artículo se describen los comandos de control de tabla externos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 580f675360b96d56d43e1100cbba97d09a95c945
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227713"
---
# <a name="external-table-control-commands"></a>Comandos de control de tabla externa

Vea [tablas externas](../query/schema-entities/externaltables.md) para obtener información general sobre las tablas externas. 

## <a name="common-external-tables-control-commands"></a>Comandos de control comunes de tablas externas

Los siguientes comandos son relevantes para _cualquier_ tabla externa (de cualquier tipo).

## <a name="show-external-tables"></a>. Mostrar tablas externas

* Devuelve todas las tablas externas de la base de datos (o una tabla externa específica).
* Requiere el [permiso monitor de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show` `external` `tables`

`.show``external` `table` *TableName*

**Salida**

| Parámetro de salida | Tipo   | Descripción                                                         |
|------------------|--------|---------------------------------------------------------------------|
| TableName        | string | Nombre de la tabla externa                                             |
| TableType        | string | Tipo de tabla externa                                              |
| Carpeta           | string | Carpeta de la tabla                                                     |
| DocString        | string | Cadena que documenta la tabla                                       |
| Propiedades       | string | Propiedades serializadas de JSON de la tabla (específicas del tipo de tabla) |


**Ejemplos:**

```kusto
.show external tables
.show external table T
```

| TableName | TableType | Carpeta         | DocString | Propiedades |
|-----------|-----------|----------------|-----------|------------|
| T         | Blob      | ExternalTables | Docs      | {}         |


## <a name="show-external-table-schema"></a>. Mostrar esquema de tabla externa

* Devuelve el esquema de la tabla externa, como JSON o CSL. 
* Requiere el [permiso monitor de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show``external` `table` *TableName* `schema` `as` ( `json`  |  `csl` )

`.show``external` `table` *TableName*`cslschema`

**Salida**

| Parámetro de salida | Tipo   | Descripción                        |
|------------------|--------|------------------------------------|
| TableName        | string | Nombre de la tabla externa            |
| Schema           | string | El esquema de tabla en formato JSON |
| DatabaseName     | string | Nombre de la base de datos de la tabla             |
| Carpeta           | string | Carpeta de la tabla                    |
| DocString        | string | Cadena que documenta la tabla      |

**Ejemplos:**

```kusto
.show external table T schema as JSON
```

```kusto
.show external table T schema as CSL
.show external table T cslschema
```

**Salida:**

*JSON*

| TableName | Schema    | DatabaseName | Carpeta         | DocString |
|-----------|----------------------------------|--------------|----------------|-----------|
| T         | {"Name": "ExternalBlob",<br>"Carpeta": "ExternalTables",<br>"DocString": "docs",<br>"OrderedColumns": [{"Name": "x", "type": "System. Int64", "CslType": "Long", "DocString": ""}, {"Name": "s", "type": "System. String", "CslType": "String", "DocString": ""}]} | DB           | ExternalTables | Docs      |


*CSL*

| TableName | Schema          | DatabaseName | Carpeta         | DocString |
|-----------|-----------------|--------------|----------------|-----------|
| T         | x:long,s:string | DB           | ExternalTables | Docs      |

## <a name="drop-external-table"></a>. quitar tabla externa

* Quita una tabla externa 
* No se puede restaurar la definición de la tabla externa después de esta operación
* Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:**  

`.drop``external` `table` *TableName*

**Salida**

Devuelve las propiedades de la tabla quitada. Vea [. Mostrar tablas externas](#show-external-tables).

**Ejemplos:**

```kusto
.drop external table ExternalBlob
```

| TableName | TableType | Carpeta         | DocString | Schema       | Propiedades |
|-----------|-----------|----------------|-----------|-----------------------------------------------------|------------|
| T         | Blob      | ExternalTables | Docs      | [{"Name": "x", "CslType": "Long"},<br> {"Name": "s", "CslType": "String"}] | {}         |


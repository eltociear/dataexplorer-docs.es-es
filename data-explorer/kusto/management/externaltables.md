---
title: 'Comandos de control general de tabla externa de Kusto: Azure Explorador de datos'
description: En este artículo se describen los comandos de control de tabla externos generales
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/26/2020
ms.openlocfilehash: 293ee468f31fafafdf08da1632c93b04b0a8adf2
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84329017"
---
# <a name="external-table-general-control-commands"></a>Comandos de control general de tabla externa

Vea [tablas externas](../query/schema-entities/externaltables.md) para obtener información general sobre las tablas externas. Los siguientes comandos son relevantes para _cualquier_ tabla externa (de cualquier tipo).

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
| T         | Blob      | ExternalTables | Documentos      | {}         |


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
| T         | {"Name": "ExternalBlob",<br>"Carpeta": "ExternalTables",<br>"DocString": "docs",<br>"OrderedColumns": [{"Name": "x", "type": "System. Int64", "CslType": "Long", "DocString": ""}, {"Name": "s", "type": "System. String", "CslType": "String", "DocString": ""}]} | DB           | ExternalTables | Documentos      |


*CSL*

| TableName | Schema          | DatabaseName | Carpeta         | DocString |
|-----------|-----------------|--------------|----------------|-----------|
| T         | x:long,s:string | DB           | ExternalTables | Documentos      |

## <a name="drop-external-table"></a>. quitar tabla externa

* Quita una tabla externa 
* No se puede restaurar la definición de la tabla externa después de esta operación
* Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:**  

`.drop``external` `table` *TableName*

**Salida**

Devuelve las propiedades de la tabla quitada. Para obtener más información, vea [. Mostrar tablas externas](#show-external-tables).

**Ejemplos:**

```kusto
.drop external table ExternalBlob
```

| TableName | TableType | Carpeta         | DocString | Schema       | Propiedades |
|-----------|-----------|----------------|-----------|-----------------------------------------------------|------------|
| T         | Blob      | ExternalTables | Documentos      | [{"Name": "x", "CslType": "Long"},<br> {"Name": "s", "CslType": "String"}] | {}         |

## <a name="next-steps"></a>Pasos siguientes

* [Creación y modificación de tablas externas en Azure Storage o Azure Data Lake](external-tables-azurestorage-azuredatalake.md)
* [Creación y modificación de tablas SQL externas](external-sql-tables.md)

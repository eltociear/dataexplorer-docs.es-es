---
title: '. Mostrar esquema de tabla: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Mostrar el esquema de tabla en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 899e54b46dd231db0bf1272c0eb1933dad474a47
ms.sourcegitcommit: 2ebd83369f247cf6dd91709f26e4ecd873489eaa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2020
ms.locfileid: "83555022"
---
# <a name="show-table-schema"></a>.show table schema

Obtiene el esquema que se va a usar en los comandos CREATE/ALTER y en los metadatos de tabla adicionales.

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

```kusto
.show table TableName cslschema 
```

| Parámetro de salida | Tipo   | Descripción                                               |
|------------------|--------|-----------------------------------------------------------|
| TableName        | String | Nombre de la tabla.                                    |
| Schema           | String | El esquema de tabla como debe usarse para CREATE/ALTER de tabla |
| DatabaseName     | String | La base de datos a la que pertenece la tabla                   |
| Carpeta           | String | Carpeta de la tabla                                            |
| DocString        | String | DocString de la tabla                                         |


## <a name="show-table-schema-as-json"></a>. Mostrar el esquema de tabla como JSON

Obtiene el esquema en formato JSON y metadatos de tabla adicionales.

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

```kusto
.show table TableName schema as json
```

| Parámetro de salida | Tipo   | Descripción                             |
|------------------|--------|-----------------------------------------|
| TableName        | String | Nombre de la tabla.                   |
| Schema           | String | Esquema de tabla en formato JSON         |
| DatabaseName     | String | La base de datos a la que pertenece la tabla |
| Carpeta           | String | Carpeta de la tabla                          |
| DocString        | String | DocString de la tabla                       |

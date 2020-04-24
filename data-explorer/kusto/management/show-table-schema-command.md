---
title: 'Esquema de tabla .show: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el esquema de tabla .show en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 9223baa0242cc936b25a7d3293d102cb3966ed53
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519778"
---
# <a name="show-table-schema"></a>Esquema de tabla .show

Obtiene el esquema que se usará en los comandos create/alter y en los metadatos de tabla adicionales.

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .

```
.show table TableName cslschema 
```
| Parámetro de salida | Tipo   | Descripción                                               |
|------------------|--------|-----------------------------------------------------------|
| TableName        | String | Nombre de la tabla.                                    |
| Schema           | String | El esquema de tabla que se debe utilizar para la tabla create/alter |
| DatabaseName     | String | La base de datos a la que pertenece la tabla                   |
| Carpeta           | String | Carpeta de la tabla                                            |
| DocString        | String | Docstring de la mesa                                         |


## <a name="show-table-schema-as-json"></a>Esquema de tabla .show como JSON

Obtiene el esquema en formato JSON y metadatos de tabla adicionales.

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .

```
.show table TableName schema as JSON
```

| Parámetro de salida | Tipo   | Descripción                             |
|------------------|--------|-----------------------------------------|
| TableName        | String | Nombre de la tabla.                   |
| Schema           | String | El esquema de tabla en formato JSON         |
| DatabaseName     | String | La base de datos a la que pertenece la tabla |
| Carpeta           | String | Carpeta de la tabla                          |
| DocString        | String | Docstring de la mesa                       |
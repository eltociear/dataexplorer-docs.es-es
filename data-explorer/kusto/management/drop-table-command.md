---
title: Tabla .drop y tablas .drop - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las tablas .drop y las tablas .drop en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2020
ms.openlocfilehash: 5f3a488aba5a6785ceb6ad4a093c520ec0509e5e
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744743"
---
# <a name="drop-table-and-drop-tables"></a>Tabla .drop y tablas .drop

Quita una tabla (o varias tablas) de la base de datos.

Requiere [permiso](../management/access-control/role-based-authorization.md)de administrador de tabla .

> [!NOTE]
> El `.drop` `table` comando solo elimina temporalmente los datos (es decir, los datos no se pueden consultar, pero todavía se puede recuperar del almacenamiento persistente). Los artefactos de almacenamiento subyacentes `recoverability` se eliminan de forma rígida según la propiedad de la directiva de [retención](../management/retentionpolicy.md) que estaba en vigor en el momento en que se ingingieron los datos en la tabla.

**Sintaxis**

`.drop``table` *NombreTabla* `ifexists`[ ]

`.drop``tables` (*TableName1*, *TableName2*,..) [si existe]

> [!NOTE]
> Si `ifexists` se especifica, el comando no fallará si hay una tabla inexistente.

**Ejemplo**

```kusto
.drop table CustomersTable ifexists
.drop tables (ProductsTable, ContactsTable, PricesTable) ifexists
```

**Devuelve**

Este comando devuelve una lista de las tablas restantes de la base de datos. 

| Parámetro de salida | Tipo   | Descripción                             |
|------------------|--------|-----------------------------------------|
| TableName        | String | Nombre de la tabla.                  |
| DatabaseName     | String | La base de datos a la que pertenece la tabla. |
---
title: 'Tabla .create: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la tabla .create en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/06/2020
ms.openlocfilehash: 1c84b9b6cec5a5bb7ab620871f59c188bf71cef4
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744240"
---
# <a name="create-table"></a>.create table

Crea una nueva tabla vacía.

El comando debe ejecutarse en el contexto de una base de datos específica.

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .

**Sintaxis**

`.create``table` *TableName* ([columnName:columnType], ...)  [`with` `(``docstring` `=` [ *Documentación*`,` `folder` `=` ] `)`[ *NombreCarpeta*] ]

Si la tabla ya existe, el comando devolverá el éxito.

**Ejemplo** 

```kusto
.create table MyLogs ( Level:string, Timestamp:datetime, UserId:string, TraceId:string, Message:string, ProcessId:int32 ) 
```
 
**Salida de retorno**

Devuelve el esquema de la tabla en formato JSON, igual que:

```kusto
.show table MyLogs schema as json
```

> [!NOTE]
> Para crear varias tablas, utilice el comando [.create tables](/create-tables.md) para obtener un mejor rendimiento y una menor carga en el clúster.

## <a name="create-merge-table"></a>Tabla .create-merge

Crea una nueva tabla o extiende una tabla existente. 

El comando debe ejecutarse en el contexto de una base de datos específica. 

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .

**Sintaxis**

`.create-merge``table` *TableName* ([columnName:columnType], ...)  [`with` `(``docstring` `=` [ *Documentación*`,` `folder` `=` ] `)`[ *NombreCarpeta*] ]

Si la tabla no existe, funciona exactamente como el comando ".create table".

Si la tabla T existe y envía un comando<columns specification>".create-merge table T ( )", entonces:

* Cualquier columna <columns specification> que no existiera anteriormente en T se agregará al final del esquema de T.
* Cualquier columna en T <columns specification> que no esté en no se eliminará de T.
* Cualquier columna <columns specification> que exista en T, pero con un tipo de datos diferente hará que el comando falle.

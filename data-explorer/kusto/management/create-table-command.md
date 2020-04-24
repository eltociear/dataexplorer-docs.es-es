---
title: '. CREATE TABLE: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. CREATE TABLE en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/06/2020
ms.openlocfilehash: 25554b5485562179d849e846fc5e71c587815e86
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108429"
---
# <a name="create-table"></a>.create table

Crea una nueva tabla vacía.

El comando debe ejecutarse en el contexto de una base de datos específica.

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.create``table` *TableName* ([ColumnName: columnType],...)  [`with` `(`[`docstring` `=` *FolderName* *Documentation*`,` Documentación] [ `folder` NombreCarpeta] `)`] `=`

Si la tabla ya existe, el comando devolverá Success.

**Ejemplo** 

```kusto
.create table MyLogs ( Level:string, Timestamp:datetime, UserId:string, TraceId:string, Message:string, ProcessId:int32 ) 
```
 
**Devolver salida**

Devuelve el esquema de la tabla en formato JSON, igual que:

```kusto
.show table MyLogs schema as json
```

> [!NOTE]
> Para crear varias tablas, use el comando [. Create tables](create-tables-command.md) para mejorar el rendimiento y reducir la carga en el clúster.

## <a name="create-merge-table"></a>. Create: combinar tabla

Crea una nueva tabla o extiende una tabla existente. 

El comando debe ejecutarse en el contexto de una base de datos específica. 

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis**

`.create-merge``table` *TableName* ([ColumnName: columnType],...)  [`with` `(`[`docstring` `=` *FolderName* *Documentation*`,` Documentación] [ `folder` NombreCarpeta] `)`] `=`

Si la tabla no existe, funciona exactamente igual que el comando ". CREATE TABLE".

Si la tabla T existe y se envía un comando ". Create-Merge Table T<columns specification>()", entonces:

* Cualquier columna de <columns specification> que no exista previamente en t se agregará al final del esquema de t.
* Cualquier columna de T que no esté en <columns specification> no se quitará de t.
* Cualquier columna de <columns specification> que exista en T, pero con un tipo de datos diferente, provocará un error en el comando.

---
title: 'Tablas .create: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen las tablas .create en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: f76c4d4a2780b58d9596537aea183d026d086fc5
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744033"
---
# <a name="create-tables"></a>.create tables

Crea nuevas tablas vacías como una operación masiva.

El comando debe ejecutarse en el contexto de una base de datos específica.

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .

**Sintaxis**

`.create``tables` *TableName1* ([columnName:columnType], ...) [`,` *TableName2* ([columnName:columnType], ...) ... ]

Si ya existe alguna tabla, el comando devolverá el éxito.
 
**Ejemplo** 

```kusto
.create tables 
  MyLogs (Level:string, Timestamp:datetime, UserId:string, TraceId:string, Message:string, ProcessId:int32),
  MyUsers (UserId:string, Name:string)
```

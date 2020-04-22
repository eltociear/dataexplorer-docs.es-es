---
title: 'Tablas .create-merge: Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las tablas .create-merge en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 408046e198710c4b825a399fcb90960411de1041
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744448"
---
# <a name="create-merge-tables"></a>Tablas .create-merge

Permite crear o ampliar los esquemas de tablas existentes en una sola operación masiva, en el contexto de una base de datos específica.

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos, así como permiso de administrador de [tabla](../management/access-control/role-based-authorization.md) para extender las tablas existentes.

**Sintaxis**

`.create-merge``tables` *TableName1* ([columnName:columnType], ...) [`,` *TableName2* ([columnName:columnType], ...) ... ]

* Se crearán tablas especificadas que no existan.
* Las tablas especificadas que ya existen tendrán sus esquemas extendidos:
    * Las columnas no existentes se agregarán al _final_ del esquema de la tabla existente.
    * Las columnas existentes que no se especifican en el comando no se quitarán del esquema de la tabla existente.
    * Las columnas existentes que se especifican con un tipo de datos diferente en el comando que en el esquema de la tabla existente dará lugar a un error (no se creará ni ampliará ninguna tabla).

**Ejemplo** 

```kusto
.create-merge tables 
  MyLogs (Level:string, Timestamp:datetime, UserId:string, TraceId:string, Message:string, ProcessId:int32),
  MyUsers (UserId:string, Name:string)
```

**Salida de retorno**

| TableName | DatabaseName  | Carpeta | DocString |
|-----------|---------------|--------|-----------|
| MyLogs    | TopComparison |        |           |
| MyUsers   | TopComparison |        |           |

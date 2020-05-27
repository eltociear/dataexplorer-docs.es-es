---
title: '. creación y combinación de tablas: Azure Explorador de datos'
description: En este artículo se describe el comando. Create-Merge tables en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 2f80ea54ece66440dc7a6b40d9d571f04bd3e26b
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011523"
---
# <a name="create-merge-tables"></a>. Create: combinar tablas

Permite crear y extender los esquemas de las tablas existentes en una única operación masiva, en el contexto de una base de datos específica.

> [!NOTE]
> Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).
> Requiere el [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para extender las tablas existentes.

**Sintaxis**

`.create-merge``tables` *TableName1* ([ColumnName: columntype],...) [ `,` *TableName2* ([ColumnName: columntype],...)...]

* Se crearán las tablas especificadas que no existen.
* Se extienden los esquemas de las tablas especificadas que ya existen.
    * Las columnas no existentes se agregarán al _final_ del esquema de la tabla existente.
    * Las columnas existentes que no se especifican en el comando no se quitarán del esquema de la tabla existente.
    * Las columnas existentes que se especifican con un tipo de datos en el comando que es diferente de la de los esquemas de la tabla existente darán lugar a un error. No se creará ni extenderá ninguna tabla.

**Ejemplo**

```kusto
.create-merge tables 
  MyLogs (Level:string, Timestamp:datetime, UserId:string, TraceId:string, Message:string, ProcessId:int32),
  MyUsers (UserId:string, Name:string)
```

**Devolver salida**

| TableName | DatabaseName  | Carpeta | DocString |
|-----------|---------------|--------|-----------|
| MyLogs    | Comparación |        |           |
| Mis usuarios   | Comparación |        |           |

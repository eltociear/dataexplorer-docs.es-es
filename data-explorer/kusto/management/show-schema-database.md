---
title: 'Esquema de bases de datos .show: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el esquema de bases de datos .show en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 212aaf428e03190226a17509c97e9acd1394d2b1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519812"
---
# <a name="show-databases-schema"></a>Esquema de bases de datos .show

Devuelve una lista plana de la estructura de las bases de datos seleccionadas con todas sus tablas y columnas en una sola tabla o objeto JSON.
Cuando se usa con una versión, la base de datos solo se devuelve si es una versión posterior a la versión proporcionada.

> [!NOTE]
> La versión sólo debe proporcionarse en formato "vMM.mm". MM representa la versión principal y mm representa la versión secundaria.

`.show``database` *DatabaseName* `schema` `details`[`if_later_than` ] [ *"Version"*] 

`.show``database` *DatabaseName* `schema` `if_later_than` [ *"Versión"*] `as``json`
 
`.show``databases` `,` DatabaseName1 ... *DatabaseName1* `(` `)` `schema` [`details` | `as` `json`]
 
`.show``databases` `,` *"Version"* *DatabaseName1* DatabaseName1 if_later_than "Versión" ... `(` `)` `schema` [`details` | `as` `json`]

**Ejemplo** 
 
La base de datos 'TestDB' tiene 1 tabla denominada 'Eventos'.

```
.show database TestDB schema 
```

**Salida del ejemplo**

|DatabaseName|TableName|ColumnName|ColumnType|IsDefaultTable|IsDefaultColumn|PrettyName|Versión
|---|---|---|---|---|---|---|--- 
|TestDB||||False|False||v.1.1       
|TestDB|Events|||True|False||       
|TestDB|Events| NOMBRE|System.String|True|False||     
|TestDB|Events| StartTime|  System.DateTime|True|False||    
|TestDB|Events| EndTime|    System.DateTime|True|False||        
|TestDB|Events| City|   System.String|True| False||     
|TestDB|Events| SessionId|  System.Int32|True|  True|| 

**Ejemplo** 
 
```
.show database TestDB schema if_later_than "v1.0" 
```
**Salida del ejemplo**

|DatabaseName|TableName|ColumnName|ColumnType|IsDefaultTable|IsDefaultColumn|PrettyName|Versión
|---|---|---|---|---|---|---|--- 
|TestDB||||False|False||v.1.1       
|TestDB|Events|||True|False||       
|TestDB|Events| NOMBRE|System.String|True|False||     
|TestDB|Events| StartTime|  System.DateTime|True|False||    
|TestDB|Events| EndTime|    System.DateTime|True|False||        
|TestDB|Events| City|   System.String|True| False||     
|TestDB|Events| SessionId|  System.Int32|True|  True||  

Dado que se proporcionó una versión inferior a la versión actual de la base de datos, se devolvió el esquema 'TestDB'. Proporcionar una versión igual o superior habría generado un resultado vacío.

**Ejemplo** 
 
```
.show database TestDB schema as json
```

**Salida del ejemplo**

```json
"{""Databases"":{""TestDB"":{""Name"":""TestDB"",""Tables"":{""Events"":{""Name"":""Events"",""DefaultColumn"":null,""OrderedColumns"":[{""Name"":""Name"",""Type"":""System.String""},{""Name"":""StartTime"",""Type"":""System.DateTime""},{""Name"":""EndTime"",""Type"":""System.DateTime""},{""Name"":""City"",""Type"":""System.String""},{""Name"":""SessionId"",""Type"":""System.Int32""}]}},""PrettyName"":null,""MajorVersion"":1,""MinorVersion"":1,""Functions"":{}}}}"
```


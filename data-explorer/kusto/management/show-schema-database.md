---
title: '. Mostrar esquema de bases de datos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Mostrar el esquema de bases de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 90a48ec3a830e754bf823712ca7016d162474a39
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616990"
---
# <a name="show-databases-schema"></a>. Mostrar esquema de bases de datos

Devuelve una lista plana de la estructura de las bases de datos seleccionadas con todas sus tablas y columnas en una sola tabla o objeto JSON.
Cuando se usa con una versión, la base de datos solo se devuelve si se trata de una versión posterior a la especificada.

> [!NOTE]
> La versión solo debe proporcionarse en formato "vMM.mm". MM representa la versión principal y los mm representan la versión secundaria.

`.show``database` *DatabaseName* Databasename `schema` [`details`] [`if_later_than` *"versión"*] 

`.show``database` *DatabaseName* Databasename `schema` [`if_later_than` *"versión"*] `as``json`
 
`.show``databases` `,` *DatabaseName1* DatabaseName1 `(` ... `)` `schema` [`details` | `as` `json`]
 
`.show``databases` *"Version"* `,` *DatabaseName1* DatabaseName1 if_later_than "version"... `(` `)` `schema` [`details` | `as` `json`]

**Ejemplo** 
 
La base de datos ' TestDB ' tiene una tabla denominada ' Events '.

```kusto
.show database TestDB schema 
```

**Salida del ejemplo**

|DatabaseName|TableName|ColumnName|ColumnType|IsDefaultTable|IsDefaultColumn|PrettyName|Versión
|---|---|---|---|---|---|---|--- 
|TestDB||||False|False||versión 1.1       
|TestDB|Events|||True|False||       
|TestDB|Events| Nombre|System.String|True|False||     
|TestDB|Events| StartTime|  System.DateTime|True|False||    
|TestDB|Events| EndTime|    System.DateTime|True|False||        
|TestDB|Events| City|   System.String|True| False||     
|TestDB|Events| SessionId|  System.Int32|True|  True|| 

**Ejemplo** 
 
```kusto
.show database TestDB schema if_later_than "v1.0" 
```
**Salida del ejemplo**

|DatabaseName|TableName|ColumnName|ColumnType|IsDefaultTable|IsDefaultColumn|PrettyName|Versión
|---|---|---|---|---|---|---|--- 
|TestDB||||False|False||versión 1.1       
|TestDB|Events|||True|False||       
|TestDB|Events| Nombre|System.String|True|False||     
|TestDB|Events| StartTime|  System.DateTime|True|False||    
|TestDB|Events| EndTime|    System.DateTime|True|False||        
|TestDB|Events| City|   System.String|True| False||     
|TestDB|Events| SessionId|  System.Int32|True|  True||  

Dado que se proporcionó una versión inferior a la versión de la base de datos actual, se devolvió el esquema ' TestDB '. Proporcionar una versión igual o superior habría generado un resultado vacío.

**Ejemplo** 
 
```kusto
.show database TestDB schema as json
```

**Salida del ejemplo**

```json
"{""Databases"":{""TestDB"":{""Name"":""TestDB"",""Tables"":{""Events"":{""Name"":""Events"",""DefaultColumn"":null,""OrderedColumns"":[{""Name"":""Name"",""Type"":""System.String""},{""Name"":""StartTime"",""Type"":""System.DateTime""},{""Name"":""EndTime"",""Type"":""System.DateTime""},{""Name"":""City"",""Type"":""System.String""},{""Name"":""SessionId"",""Type"":""System.Int32""}]}},""PrettyName"":null,""MajorVersion"":1,""MinorVersion"":1,""Functions"":{}}}}"
```


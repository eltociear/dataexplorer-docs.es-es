---
title: pack_all() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe pack_all() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3c6a22b656e28b8b7113864e0b3f9636a4fb364d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511941"
---
# <a name="pack_all"></a>pack_all()

Crea `dynamic` un objeto (bolsa de propiedades) a partir de todas las columnas de la expresión tabular.

**Sintaxis**

`pack_all()`

**Ejemplos**

Dada una tabla SmsMessages 

|SourceNumber |TargetNumber| CharsCount
|---|---|---
|555-555-1234 |555-555-1212 | 46 
|555-555-1234 |555-555-1213 | 50 
|555-555-1212 |555-555-1234 | 32 

La siguiente consulta:
```kusto
SmsMessages | extend Packed=pack_all()
``` 

Devuelve:

|TableName |SourceNumber |TargetNumber | Embalado
|---|---|---|---
|SmsMessages|555-555-1234 |555-555-1212 | "SourceNumber":"555-555-1234", "TargetNumber":"555-555-1212", "CharsCount": 46o
|SmsMessages|555-555-1234 |555-555-1213 | "SourceNumber":"555-555-1234", "TargetNumber":"555-555-1213", "CharsCount": 50o
|SmsMessages|555-555-1212 |555-555-1234 | "SourceNumber":"555-555-1212", "TargetNumber":"555-555-1234", "CharsCount": 32o
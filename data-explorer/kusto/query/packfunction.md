---
title: pack() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe pack() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a7b43be96f272ab929434f10cac910bd4072e650
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511839"
---
# <a name="pack"></a>pack()

Crea `dynamic` un objeto (bolsa de propiedades) a partir de una lista de nombres y valores.

Alias `pack_dictionary()` para funcionar.

**Sintaxis**

`pack(`*key1* `,` *value1* `,` *key2* `,` *value2*`,... )`

**Argumentos**

* Una lista alterna de claves y valores (la longitud total de la lista debe ser par)
* Todas las claves deben ser cadenas constantes no vacías

**Ejemplos**

El siguiente ejemplo devuelve `{"Level":"Information","ProcessID":1234,"Data":{"url":"www.bing.com"}}`:

```kusto
pack("Level", "Information", "ProcessID", 1234, "Data", pack("url", "www.bing.com"))
```

Tomemos 2 tablas, SmsMessages y MmsMessages:

SmsMessages de tabla 

|SourceNumber |TargetNumber| CharsCount
|---|---|---
|555-555-1234 |555-555-1212 | 46 
|555-555-1234 |555-555-1213 | 50 
|555-555-1212 |555-555-1234 | 32 

MmsMessages de la tabla 

|SourceNumber |TargetNumber| AttachmnetSize | AttachmnetType | AttachmnetName
|---|---|---|---|---
|555-555-1212 |555-555-1213 | 200 | JPEG | Pic1
|555-555-1234 |555-555-1212 | 250 | JPEG | Pic2
|555-555-1234 |555-555-1213 | 300 | png | Pic3

La siguiente consulta:
```kusto
SmsMessages 
| extend Packed=pack("CharsCount", CharsCount) 
| union withsource=TableName kind=inner 
( MmsMessages 
  | extend Packed=pack("AttachmnetSize", AttachmnetSize, "AttachmnetType", AttachmnetType, "AttachmnetName", AttachmnetName))
| where SourceNumber == "555-555-1234"
``` 

Devuelve:

|TableName |SourceNumber |TargetNumber | Embalado
|---|---|---|---
|SmsMessages|555-555-1234 |555-555-1212 | "CharsCount": 46o
|SmsMessages|555-555-1234 |555-555-1213 | "CharsCount": 50o
|MmsMessages|555-555-1234 |555-555-1212 | "AttachmnetSize": 250, "AttachmnetType": "jpeg", "AttachmnetName": "Pic2"
|MmsMessages|555-555-1234 |555-555-1213 | "AttachmnetSize": 300, "AttachmnetType": "png", "AttachmnetName": "Pic3"
---
title: Pack ()-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe el módulo () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a94629ae8f4795e28cbfb0c41f06596731cdd8d9
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717332"
---
# <a name="pack"></a>pack()

Crea un `dynamic` objeto (contenedor de propiedades) a partir de una lista de nombres y valores.

Alias para la `pack_dictionary()` función.

**Sintaxis**

`pack(`*Key1* `,` *valor1* `,` *KEY2* `,` *valor2*`,... )`

**Argumentos**

* Una lista de claves y valores alternas (la longitud total de la lista debe ser par)
* Todas las claves deben ser cadenas constantes no vacías

**Ejemplos**

El siguiente ejemplo devuelve `{"Level":"Information","ProcessID":1234,"Data":{"url":"www.bing.com"}}`:

```kusto
pack("Level", "Information", "ProcessID", 1234, "Data", pack("url", "www.bing.com"))
```

Permite tomar 2 tablas, SmsMessages y MmsMessages:

Tabla SmsMessages 

|SourceNumber |TargetNumber| CharsCount
|---|---|---
|555-555-1234 |555-555-1212 | 46 
|555-555-1234 |555-555-1213 | 50 
|555-555-1212 |555-555-1234 | 32 

Tabla MmsMessages 

|SourceNumber |TargetNumber| AttachmentSize | AttachmentType | AttachmentName
|---|---|---|---|---
|555-555-1212 |555-555-1213 | 200 | jpeg | Pic1
|555-555-1234 |555-555-1212 | 250 | jpeg | Pic2
|555-555-1234 |555-555-1213 | 300 | png | Pic3

La siguiente consulta:
```kusto
SmsMessages 
| extend Packed=pack("CharsCount", CharsCount) 
| union withsource=TableName kind=inner 
( MmsMessages 
  | extend Packed=pack("AttachmentSize", AttachmentSize, "AttachmentType", AttachmentType, "AttachmentName", AttachmentName))
| where SourceNumber == "555-555-1234"
``` 

Devuelve:

|TableName |SourceNumber |TargetNumber | Equipado
|---|---|---|---
|SmsMessages|555-555-1234 |555-555-1212 | {"CharsCount": 46}
|SmsMessages|555-555-1234 |555-555-1213 | {"CharsCount": 50}
|MmsMessages|555-555-1234 |555-555-1212 | {"AttachmentSize": 250, "AttachmentType": "JPEG", "AttachmentName": "Pic2"}
|MmsMessages|555-555-1234 |555-555-1213 | {"AttachmentSize": 300, "AttachmentType": "png", "AttachmentName": "Pic3"}

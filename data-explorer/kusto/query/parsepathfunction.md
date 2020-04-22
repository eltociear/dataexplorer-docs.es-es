---
title: parse_path() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_path() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 2267efb4e47a6d8e45733ad48dd9f7f4019c1fa8
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744663"
---
# <a name="parse_path"></a>parse_path()

Analiza una ruta `string` de [`dynamic`](./scalar-data-types/dynamic.md) acceso de archivo y devuelve un objeto que contiene las siguientes partes de la ruta de acceso: Scheme, RootPath, DirectoryPath, DirectoryName, FileName, Extension, AlternateDataStreamName.
Además de las rutas simples con ambos tipos de barras diagonales, admite rutas de acceso con\\esquemas (por ejemplo, "file://..."),\\rutas compartidas (por ejemplo, "shareddrive-users..."), rutas largas (por ejemplo, "?-C:...""), secuencias de datos alternativas (por ejemplo, "file1.exe:file2.exe")

**Sintaxis**

`parse_path(`*Camino*`)`

**Argumentos**

* *path*: cadena que representa una ruta de acceso de archivo.

**Devuelve**

Objeto de `dynamic` tipo que inculca los componentes de ruta como se indica anteriormente.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->

```kusto
datatable(p:string) 
[
    @"C:\temp\file.txt",
    @"temp\file.txt",
    "file://C:/temp/file.txt:some.exe",
    @"\\shared\users\temp\file.txt.gz",
    "/usr/lib/temp/file.txt"
]
| extend path_parts = parse_path(p)

```

|p|path_parts
|---|---
|C:-temp-archivo.txt|"Scheme":"","RootPath":"C:","DirectoryPath":"C:\\temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":""
|temp-file.txt|"Scheme":"","RootPath":"","DirectoryPath":"temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":""
|file://C:/temp/file.txt:some.exe|"Scheme":"file","RootPath":"C:","DirectoryPath":"C:/temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":"some.exe":""
|\\compartidos, usuarios, temp, archivo.txt.gz|\\\\"Scheme":"","RootPath":"","DirectoryPath":"\\usuarios\\compartidos temp","DirectoryName":"temp","Filename":"file.txt.gz","Extension":"gz","AlternateDataStreamName":""
|/usr/lib/temp/file.txt|"Scheme":"","RootPath":"","DirectoryPath":"/usr/lib/temp","DirectoryName":"temp","Filename":"file.txt","Extension":"txt","AlternateDataStreamName":""

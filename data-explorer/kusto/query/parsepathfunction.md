---
title: 'parse_path (): Explorador de datos de Azure'
description: En este artículo se describe parse_path () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 16b80c86f526cb05514577359603e9e21de80064
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83224891"
---
# <a name="parse_path"></a>parse_path()

Analiza una ruta de acceso de archivo `string` y devuelve un [`dynamic`](./scalar-data-types/dynamic.md) objeto que contiene las siguientes partes de la ruta de acceso: Scheme, RootPath, directoryPath, directoryname, FILENAME, Extension, AlternateDataStreamName.
Además de las rutas de acceso simples con ambos tipos de barras diagonales, admite rutas de acceso con esquemas (por ejemplo, "File://..."), rutas de acceso compartidas (por ejemplo \\ , "shareddrive\users..."), rutas de acceso largas (por ejemplo, " \\ ? \c:..." "), secuencias de datos alternativas (por ejemplo," archivo1

**Sintaxis**

`parse_path(`*camino*`)`

**Argumentos**

* *path*: una cadena que representa una ruta de acceso de archivo.

**Devuelve**

Objeto de tipo `dynamic` que incluye los componentes de la ruta de acceso tal y como se indicó anteriormente.

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
|C:\temp\file.txt|{"Scheme": "", "RootPath": "C:", "DirectoryPath": "C: \\ Temp", "directoryname": "Temp", "filename": "File. txt", "Extension": "txt", "AlternateDataStreamName": ""}
|temp\file.txt|{"Scheme": "", "RootPath": "", "DirectoryPath": "Temp", "DirectoryName": "Temp", "filename": "File. txt", "Extension": "txt", "AlternateDataStreamName": ""}
|file://C:/temp/file.txt:some.exe|{"Scheme": "File", "RootPath": "C:", "DirectoryPath": "C:/Temp", "DirectoryName": "Temp", "filename": "File. txt", "Extension": "txt", "AlternateDataStreamName": "Some. exe"}
|\\shared\users\temp\file.txt.gz|{"Scheme": "", "RootPath": "", "DirectoryPath": " \\ \\ usuarios compartidos \\ \\ Temp "," directoryname ":" Temp "," filename ":" File. txt. gz "," Extension ":" gz "," AlternateDataStreamName ":" "}
|/usr/lib/temp/file.txt|{"Scheme": "", "RootPath": "", "DirectoryPath": "/usr/lib/Temp", "DirectoryName": "Temp", "filename": "File. txt", "Extension": "txt", "AlternateDataStreamName": ""}

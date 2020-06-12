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
ms.openlocfilehash: ba74b7c1e78d568cc34845d56dc9768f2628192f
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717383"
---
# <a name="parse_path"></a>parse_path()

Analiza una ruta de acceso de archivo `string` y devuelve un [`dynamic`](./scalar-data-types/dynamic.md) objeto que contiene las siguientes partes de la ruta de acceso:
* Scheme
* RootPath
* DirectoryPath
* DirectoryName
* FileName
* Comprobación de actualización
* AlternateDataStreamName

Además de las rutas de acceso simples con ambos tipos de barras diagonales, la función admite rutas de acceso con:
* Esquemas. Por ejemplo, "file://..."
* Rutas de acceso compartidas. Por ejemplo, " \\ shareddrive\users..."
* Rutas de acceso largas. Por ejemplo, " \\ ? \c:..." "
* Flujos de datos alternativos. Por ejemplo, "file1.exe:file2.exe"

**Sintaxis**

`parse_path(`*path*`)`

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
|C:\temp\file.txt|{"Scheme": "", "RootPath": "C:", "DirectoryPath": "C: \\ Temp", "directoryname": "Temp", "filename": "file.txt", "Extension": "txt", "AlternateDataStreamName": ""}
|temp\file.txt|{"Scheme": "", "RootPath": "", "DirectoryPath": "Temp", "DirectoryName": "Temp", "filename": "file.txt", "Extension": "txt", "AlternateDataStreamName": ""}
|file.txt:some.exe file://C:/temp/|{"Scheme": "File", "RootPath": "C:", "DirectoryPath": "C:/Temp", "DirectoryName": "Temp", "filename": "file.txt", "Extension": "txt", "AlternateDataStreamName": "some.exe"}
|\\shared\users\temp\file.txt. gz|{"Scheme": "", "RootPath": "", "DirectoryPath": " \\ \\ usuarios compartidos \\ \\ Temp "," directoryname ":" Temp "," filename ":" file.txt. gz "," Extension ":" gz "," AlternateDataStreamName ":" "}
|file.txt/usr/lib/Temp/|{"Scheme": "", "RootPath": "", "DirectoryPath": "/usr/lib/Temp", "DirectoryName": "Temp", "filename": "file.txt", "Extension": "txt", "AlternateDataStreamName": ""}

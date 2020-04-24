---
title: alter-merge tabla column-docstrings - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe la alternancia de columnas y docstrings de la tabla de combinación de alter en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 75d298f35a215af5da443f673278e4a252c24cc9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522396"
---
# <a name="alter-merge-table-column-docstrings"></a>alter-merge tabla columna-docstrings

Establece `docstring` la propiedad de una o varias columnas de la tabla especificada. Las columnas no establecidas explícitamente **conservan** su valor existente para esta propiedad, si tienen uno.

Para modificar la tabla column-docstring, consulte [a continuación](#alter-table-column-docstrings).

**Sintaxis**

`.alter-merge``table` *NombreTabla* `column-docstring` *Col1* `:` `:` `,` *Col2* *Docstring1* *Docstring2*Col1 Docstring1 [ Col2 Docstring2 ]... `(``)`

**Ejemplo** 

```
.alter-merge table Table1 column-docstrings (Column1:"DocString1", Column2:"DocString2")
```

## <a name="alter-table-column-docstrings"></a>alterar tabla columna-docstrings

Establece `docstring` la propiedad de una o varias columnas de la tabla especificada. Las columnas no establecidas explícitamente tendrán esta propiedad **quitada.**

**Sintaxis**

`.alter``table` *NombreTabla* `column-docstring` *Col1* `:` `:` `,` *Col2* *Docstring1* *Docstring2*Col1 Docstring1 [ Col2 Docstring2 ]... `(``)`

**Ejemplo** 

```
.alter table Table1 column-docstrings (Column1:"DocString1", Column2:"DocString2")
```
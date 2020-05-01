---
title: columna de tabla Alter-Merge-docstrings-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe Alter-Merge table column-docstrings en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7dd36181be1140d3960369b1c8a5284ed55e48f5
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616497"
---
# <a name="alter-merge-table-column-docstrings"></a>columna de tabla Alter-Merge-docstrings

Establece la `docstring` propiedad de una o más columnas de la tabla especificada. Las columnas no establecidas explícitamente **conservan** su valor existente para esta propiedad, si tienen una.

Para alter table column-DocString, vea [a continuación](#alter-table-column-docstrings).

**Sintaxis**

`.alter-merge``table` *TableName* `:` *Docstring1* *Col1* `:` *Docstring2*col1 Docstring1 [`,` Col2 Docstring2]... *Col2* `column-docstring` `(``)`

**Ejemplo** 

```kusto
.alter-merge table Table1 column-docstrings (Column1:"DocString1", Column2:"DocString2")
```

## <a name="alter-table-column-docstrings"></a>alter table column-docstrings

Establece la `docstring` propiedad de una o más columnas de la tabla especificada. Las columnas no establecidas explícitamente tendrán esta propiedad **quitada**.

**Sintaxis**

`.alter``table` *TableName* `:` *Docstring1* *Col1* `:` *Docstring2*col1 Docstring1 [`,` Col2 Docstring2]... *Col2* `column-docstring` `(``)`

**Ejemplo** 

```kusto
.alter table Table1 column-docstrings (Column1:"DocString1", Column2:"DocString2")
```

---
title: . ALTER FUNCTION DocString-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe. ALTER FUNCTION DocString en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: d590e93a6772483aba6b9580b26490eb2fe5ec08
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617857"
---
# <a name="alter-function-docstring"></a>. Alter (función) DocString

Modifica el valor de DocString de una función existente.

`.alter``function` *FunctionName* `docstring` *Documentación* de functionname

> [!NOTE]
> * Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md)
> * El [usuario de base de datos](../management/access-control/role-based-authorization.md) que creó originalmente la función tiene permiso para modificar la función. 
> * Si la función no existe, se devuelve un error. Para crear una nueva función, vea [. Create (función](create-function.md) )

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas por una expresión de CSL válida que se evalúa cuando se invoca la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de la interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Descripción de la función para la interfaz de usuario.

**Ejemplo** 

```kusto
.alter function MyFunction1 docstring "Updated docstring"
```
    
|Nombre |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(malimit: Long)| {StormEvents &#124; limitar el límite}|MyFolder|Actualizado DocString|
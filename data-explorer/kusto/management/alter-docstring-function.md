---
title: .alter function docstring - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe la función .alter docstring en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 25f6dac67f65add545f7215b44daafb0257a8375
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522583"
---
# <a name="alter-function-docstring"></a>Docstring de la función .alter

Altera el valor DocString de una función existente.

`.alter``function` *Documentación de FunctionName* `docstring` *Documentation*

> [!NOTE]
> * Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos
> * El usuario de base de [datos](../management/access-control/role-based-authorization.md) que creó originalmente la función puede modificar la función. 
> * Si la función no existe, se devuelve un error. Para crear una nueva función, consulte [la función .create](create-function.md)

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Una descripción de la función para fines de interfaz de usuario.

**Ejemplo** 

```
.alter function MyFunction1 docstring "Updated docstring"
```
    
|NOMBRE |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(myLimit: long)| •StormEvents &#124; limitar myLimit|MyFolder|Docstring actualizado|
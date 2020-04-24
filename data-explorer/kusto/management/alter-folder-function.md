---
title: 'Carpeta de la función .alter: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la carpeta de funciones .alter en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 140991c723ebdd12fa17000ea845adbbfdd27771
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522549"
---
# <a name="alter-function-folder"></a>Carpeta de funciones .alter

Altera el valor de Carpeta de una función existente.

`.alter``function` *Carpeta* *FunctionName* `folder`

> [!NOTE]
> * Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos
> * El usuario de base de [datos](../management/access-control/role-based-authorization.md) que creó originalmente la función puede modificar la función. 
> * Si la función no existe, se devuelve un error. Para crear una nueva función, [la función .create](create-function.md)

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) Let instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta que se usa para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Una descripción de la función para fines de interfaz de usuario.

**Ejemplo** 

```
.alter function MyFunction1 folder "Updated Folder"
```
    
|NOMBRE |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(myLimit: long)| •StormEvents &#124; limitar myLimit|Carpeta actualizada|Algunos DocString|

```
.alter function MyFunction1 folder @"First Level\Second Level"
```
    
|NOMBRE |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(myLimit: long)| •StormEvents &#124; limitar myLimit|Primer Nivel-Segundo Nivel|Algunos DocString|
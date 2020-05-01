---
title: . ALTER FUNCTION Folder-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe la carpeta de la función Alter en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 6becb5e29fd5771e1027c824b5a3539ed3c33b88
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617840"
---
# <a name="alter-function-folder"></a>. Alter (carpeta de función)

Modifica el valor de la carpeta de una función existente.

`.alter``function` *FunctionName* `folder` *Carpeta* functionname

> [!NOTE]
> * Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md)
> * El [usuario de base de datos](../management/access-control/role-based-authorization.md) que creó originalmente la función tiene permiso para modificar la función. 
> * Si la función no existe, se devuelve un error. Para crear una nueva función, [. Create (función](create-function.md) )

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros que requiere la función.
|Body  |String |(Cero o más) Instrucciones Let seguidas de una expresión de CSL válida que se evalúa cuando se invoca la función.
|Carpeta|String|Una carpeta que se usa para la categorización de funciones de la interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Descripción de la función para la interfaz de usuario.

**Ejemplo** 

```kusto
.alter function MyFunction1 folder "Updated Folder"
```
    
|Nombre |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(malimit: Long)| {StormEvents &#124; limitar el límite}|Carpeta actualizada|Algunos DocString|

```kusto
.alter function MyFunction1 folder @"First Level\Second Level"
```
    
|Nombre |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(malimit: Long)| {StormEvents &#124; limitar el límite}|Primer nivel de Level\Second|Algunos DocString|
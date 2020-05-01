---
title: '. ALTER FUNCTION: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la función. Alter en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: d8248fdd9428df11a8e77316eec621102ddff9d6
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617806"
---
# <a name="alter-function"></a>.alter function

Modifica una función existente y la almacena en los metadatos de la base de datos.
Las reglas para los tipos de parámetros y las instrucciones de CSL [ `let` ](../query/letstatement.md)son las mismas que para las instrucciones.

**Sintaxis**

```kusto
.alter function [with (docstring = '<description>', folder='<name>', skipvalidation='true')] [FunctionName] ([paramName:paramType], ...) { CSL-statement }
```
    
|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función.
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas por una expresión de CSL válida que se evalúa cuando se invoca la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de la interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Descripción de la función para la interfaz de usuario.

> [!NOTE]
> * Si la función no existe, se devuelve un error. Para crear una nueva función, vea [. Create (función](create-function.md) )
> * Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md)
> * El [usuario de base de datos](../management/access-control/role-based-authorization.md) que creó originalmente la función tiene permiso para modificar la función. 
> * No todos los tipos Kusto se admiten en `let` las instrucciones. Los tipos admitidos son: String, Long, DateTime, TimeSpan y Double.
> * Use `skipvalidation` para omitir la validación semántica de la función. Esto resulta útil cuando se crean funciones en un orden incorrecto y se crea antes la tecla F1 que usa F2.
 
**Ejemplo** 

```kusto
.alter function
with (docstring = 'Demo function with parameter', folder='MyFolder')
 MyFunction2(myLimit: long)  {StormEvents | limit myLimit}
``` 
    
|Nombre |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(malimit: Long)| {StormEvents &#124; limitar el límite}|MyFolder|Función demo con el parámetro|

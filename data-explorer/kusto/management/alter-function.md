---
title: 'Función .alter: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la función .alter en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 3fb7b3aab9d140d5f3660ef1384bf54efa9cd825
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522464"
---
# <a name="alter-function"></a>Función .alter

Altera una función existente y la almacena dentro de los metadatos de la base de datos.
Las reglas para los tipos de parámetro y [ `let` ](../query/letstatement.md)las instrucciones CSL son las mismas que para las instrucciones .

**Sintaxis**

```
.alter function [with (docstring = '<description>', folder='<name>', skipvalidation='true')] [FunctionName] ([paramName:paramType], ...) { CSL-statement }
```
    
|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función.
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Una descripción de la función para fines de interfaz de usuario.

> [!NOTE]
> * Si la función no existe, se devuelve un error. Para crear una nueva función, consulte [la función .create](create-function.md)
> * Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos
> * El usuario de base de [datos](../management/access-control/role-based-authorization.md) que creó originalmente la función puede modificar la función. 
> * No todos los tipos de `let` Kusto se admiten en instrucciones. Los tipos admitidos son: string, long, datetime, timespan y double.
> * Se `skipvalidation` utiliza para omitir la validación semántica de la función. Esto es útil cuando las funciones se crean en un orden incorrecto y F1 que utiliza F2 se crea anteriormente.
 
**Ejemplo** 

```
.alter function
with (docstring = 'Demo function with parameter', folder='MyFolder')
 MyFunction2(myLimit: long)  {StormEvents | limit myLimit}
``` 
    
|NOMBRE |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction2 |(myLimit: long)| •StormEvents &#124; limitar myLimit|MyFolder|Función de demostración con parámetro|
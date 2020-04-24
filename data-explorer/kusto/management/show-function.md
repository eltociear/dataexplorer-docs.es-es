---
title: 'Funciones .show: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen las funciones .show en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a7acfdb96570b067b0461f5a788b55fc8274c03f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519846"
---
# <a name="show-functions"></a>Funciones .show

Enumera todas las funciones almacenadas en la base de datos seleccionada actualmente.
Para devolver solo una función específica, consulte [la función .show](#show-function).

```
.show functions
```

Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .
 
|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Una descripción de la función para fines de interfaz de usuario.
 
**Ejemplo de salida** 

|NOMBRE |Parámetros|Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction1 |() | Límite de &#124; de StormEvents 100o|MyFolder|Función de demostración simple|
|MyFunction2 |(myLimit: long)| •StormEvents &#124; limitar myLimit|MyFolder|Función de demostración con parámetro|
|MyFunction3 |() | • StormEvents(100) ?|MyFolder|Función que llama a otra función||

## <a name="show-function"></a>Función .show

```
.show function MyFunc1
```

Enumera los detalles de una función almacenada específica. Para obtener una lista de **todas las** funciones, consulte [Funciones .show](#show-functions).

**Sintaxis**

`.show``function` *FunctionName*

**Salida**

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función
|DocString|String|Una descripción de la función para fines de interfaz de usuario.
 
> [!NOTE] 
> * Si la función no existe, se devuelve un error.
> * Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .
 
**Ejemplo** 

```
.show function MyFunction1 
```
    
|NOMBRE |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction1 |() | Límite de &#124; de StormEvents 100o|MyFolder|Función de demostración simple
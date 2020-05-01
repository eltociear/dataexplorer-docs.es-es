---
title: '. Mostrar funciones: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Mostrar funciones en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: aa81df5ca89c1a7dc523d7799050b8a7ad3adaba
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617335"
---
# <a name="show-functions"></a>. Mostrar funciones

Muestra todas las funciones almacenadas en la base de datos seleccionada actualmente.
Para devolver solo una función concreta, vea [. Show (función](#show-function)).

```kusto
.show functions
```

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).
 
|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas por una expresión de CSL válida que se evalúa cuando se invoca la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de la interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Descripción de la función para la interfaz de usuario.
 
**Ejemplo de salida** 

|Nombre |Parámetros|Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction1 |() | {StormEvents &#124; Limit 100}|MyFolder|Función de demostración sencilla|
|MyFunction2 |(malimit: Long)| {StormEvents &#124; limitar el límite}|MyFolder|Función demo con el parámetro|
|MyFunction3 |() | {StormEvents (100)}|MyFolder|Función que llama a otra función||

## <a name="show-function"></a>.show function

```kusto
.show function MyFunc1
```

Muestra los detalles de una función almacenada concreta. Para obtener una lista de **todas** las funciones, vea [. Mostrar funciones](#show-functions).

**Sintaxis**

`.show``function` *Nombrefunción*

**Salida**

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas por una expresión de CSL válida que se evalúa cuando se invoca la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de la interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función
|DocString|String|Descripción de la función para la interfaz de usuario.
 
> [!NOTE] 
> * Si la función no existe, se devuelve un error.
> * Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).
 
**Ejemplo** 

```kusto
.show function MyFunction1 
```
    
|Nombre |Parámetros |Body|Carpeta|DocString
|---|---|---|---|---
|MyFunction1 |() | {StormEvents &#124; Limit 100}|MyFolder|Función de demostración sencilla

---
title: 'Función .drop: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la función .drop en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 8930f7333ff18fad0d5b3dbbebe9328f8bf7a0b9
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744778"
---
# <a name="drop-function"></a>.drop function

Quita una función de la base de datos.
Para quitar varias funciones de la base de datos, consulte [Funciones .drop](#drop-functions).

**Sintaxis**

`.drop``function` *FunctionName* `ifexists`[ ]

* `ifexists`: Si se especifica, modifica el comportamiento del comando para que no falle para una función inexistente.

> [!NOTE]
> * Requiere [permiso](../management/access-control/role-based-authorization.md)de administrador de base de datos .
> * El usuario de base de [datos](../management/access-control/role-based-authorization.md) que creó originalmente la función puede modificar la función.  
    
|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función que se eliminó
 
**Ejemplo** 

```kusto
.drop function MyFunction1
```

## <a name="drop-functions"></a>Funciones .drop

Quita varias funciones de la base de datos.

**Sintaxis**

`.drop``functions` (*FunctionName1*, *FunctionName2*,..) [si existe]

**Devuelve**

Este comando devuelve una lista de las funciones restantes de la base de datos.

|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Una descripción de la función para fines de interfaz de usuario.

Requiere permiso de administrador de [función](../management/access-control/role-based-authorization.md).

**Ejemplo** 
 
```kusto
.drop functions (Function1, Function2, Function3) ifexists
```

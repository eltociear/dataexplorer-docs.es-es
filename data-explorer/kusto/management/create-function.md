---
title: 'Función .create: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la función .create en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bc2caa5dcc886964b42bf1915bf3a003f2d32c7a
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744462"
---
# <a name="create-function"></a>.create function

Crea una función almacenada, que es una función de [ `let` instrucción](../query/letstatement.md) reutilizable con el nombre especificado. La definición de función se conserva con los metadatos de la base de datos.

Las funciones pueden llamar a otras funciones `let` (no se admite recursiva) y se permiten instrucciones como parte del cuerpo de la *función*. Ver [ `let` declaraciones](../query/letstatement.md).

Las reglas para los tipos de parámetro y [ `let` ](../query/letstatement.md)las instrucciones CSL son las mismas que para las instrucciones .
    
**Sintaxis**

`.create``function` [`ifnotexists`]`with` `(``docstring` `=` [ *Documentation*]`,` `folder` `=` `)`[`,` `skipvalidation` `=` *FolderName*] `)`] [ 'true']`,` ] *FunctionName* `(` *ParamName* `:` *ParamType* [ ...] `)` `{` *FunctionBody*`}`

**Salida**    
    
|Parámetro de salida |Tipo |Descripción
|---|---|--- 
|Nombre  |String |El nombre de la función. 
|Parámetros  |String |Los parámetros requeridos por la función.
|Body  |String |(Cero o más) `let` instrucciones seguidas de una expresión CSL válida que se evalúa tras la invocación de la función.
|Carpeta|String|Carpeta utilizada para la categorización de funciones de interfaz de usuario. Este parámetro no cambia la forma en que se invoca la función.
|DocString|String|Una descripción de la función para fines de interfaz de usuario.

> [!NOTE]
> * Si la función ya existe:
>    * Si `ifnotexists` se especifica flag, se omite el comando (no se aplica ningún cambio).
>    * Si `ifnotexists` no se especifica flag, se devuelve un error.
>    * Para modificar una función existente, consulte la [función .alter](alter-function.md)
> * Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .
> * No todos los tipos `let` de datos se admiten en instrucciones. Los tipos admitidos son: booleano, string, long, datetime, timespan, double y dynamic.
> * Utilice 'skipvalidation' para omitir la validación semántica de la función. Esto es útil cuando las funciones se crean en un orden incorrecto y F1 que utiliza F2 se crea anteriormente.

**Ejemplos** 

```kusto
.create function 
with (docstring = 'Simple demo function', folder='Demo')
MyFunction1()  {StormEvents | limit 100}
```

|NOMBRE|Parámetros|Body|Carpeta|DocString|
|---|---|---|---|---|
|MyFunction1|()|Límite de &#124; de StormEvents 100o|Demostración|Función de demostración simple|

```kusto
.create function
with (docstring = 'Demo function with parameter', folder='Demo')
 MyFunction2(myLimit: long)  {StormEvents | limit myLimit}
```

|NOMBRE|Parámetros|Body|Carpeta|DocString|
|---|---|---|---|---|
|MyFunction2|(myLimit:long)|•StormEvents &#124; limitar myLimit|Demostración|Función de demostración con parámetro|

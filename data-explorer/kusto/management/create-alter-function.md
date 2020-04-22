---
title: 'Función .create-or-alter: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la función .create-or-alter en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 9e9c24f7fda44d6c44b8f78d8622b525268a341a
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744267"
---
# <a name="create-or-alter-function"></a>.create-or-alter function

Crea una función almacenada o modifica una función existente y la almacena dentro de los metadatos de la base de datos.

```kusto
.create-or-alter function [with (docstring = '<description>', folder='<name>')] [FunctionName] ([paramName:paramType], ...) { CSL-statement }
```

Si la función con el *FunctionName* proporcionado no existe en los metadatos de la base de datos, el comando crea una nueva función. Si la función ya existe, se cambiará esa función.

**Ejemplo**

```kusto
.create-or-alter function  with (docstring = 'Demo function with parameter', folder='MyFolder') TestFunction(myLimit:int)
{
    StormEvents | take myLimit 
} 
```

|NOMBRE|Parámetros|Body|Carpeta|DocString|
|---|---|---|---|---|
|TestFunction|(myLimit:int)|• StormEvents &#124; tomar myLimit ?|MyFolder|Función de demostración con parámetro|

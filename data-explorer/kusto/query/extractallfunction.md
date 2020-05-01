---
title: extract_all ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe extract_all () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 94bafd3890f57a1379440c5cced3fa349f9055c5
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616429"
---
# <a name="extract_all"></a>extract_all()

Obtiene todas las coincidencias de una [expresión regular](./re2.md) de una cadena de texto.

Opcionalmente, se puede recuperar un subconjunto de grupos coincidentes.

```kusto
print extract_all(@"(\d+)", "a set of numbers: 123, 567 and 789") // results with the dynamic array ["123", "567", "789"]
```

**Sintaxis**

`extract_all(`*regex*`,` *texto* regex [*captureGroups*`,`]`)`

**Argumentos**

* *Regex*: una [expresión regular](./re2.md). La expresión regular debe tener al menos un grupo de captura y menor o igual que 16 grupos de captura.
* *captureGroups*: (opcional). Constante de matriz dinámica que indica el grupo de captura que se va a extraer. Los valores válidos son de 1 a número de grupos de captura en la expresión regular. También se permiten los grupos de captura con nombre (consulte la sección ejemplos).
* *Text*: un `string` que se va a buscar.

**Devuelve**

Si *Regex* encuentra una coincidencia en el *texto*: devuelve una matriz dinámica que incluye todas las coincidencias en los grupos de capturas indicados *captureGroups* (o todos los grupos de captura de la *expresión regular*).
Si el número de *captureGroups* es 1: la matriz devuelta tiene una única dimensión de valores coincidentes.
Si el número de *captureGroups* es mayor que 1: la matriz devuelta es una colección bidimensional de coincidencias de varios valores por selección de *captureGroups* (o todos los grupos de captura presentes en el *Regex* si se omite *captureGroups* ) 

Si no hay ninguna coincidencia: `null`. 

**Ejemplos**

### <a name="extracting-single-capture-group"></a>Extraer un solo grupo de captura
En el ejemplo siguiente se devuelve la representación de bytes hexadecimales (dos dígitos hexadecimales) del GUID.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"([\da-f]{2})", Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|["82", "B8", "es", "2D", "DF", "A7", "4B", "D1", "8F", "63", "24", "ad", "26", "D3", "14", "49"]|

### <a name="extracting-several-capture-groups"></a>Extraer varios grupos de capturas 
En el ejemplo siguiente se usa una expresión regular con 3 grupos de captura para dividir cada parte del GUID en la primera letra, la última letra y cualquier elemento del centro.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8", "2b8be2", "d"], ["d", "FA", "7"], ["4", "BD", "1"], ["8", "F6", "3"], ["2", "4ad26d3144", "9"]]|

### <a name="extracting-subset-of-capture-groups"></a>Extraer subconjunto de grupos de capturas

En el ejemplo siguiente se muestra cómo seleccionar un subconjunto de grupos de captura: en este caso, la expresión regular coincide con la primera letra, la última letra y todo el resto, mientras que el parámetro *captureGroups* se usa para seleccionar solo la primera y la última parte. 

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", dynamic([1,3]), Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8", "d"], ["d", "7"], ["4", "1"], ["8", "3"], ["2", "9"]]|


### <a name="using-named-capture-groups"></a>Usar grupos de captura con nombre

Puede emplear grupos de capturas con nombre de RE2 en extract_all (). En el ejemplo siguiente: *captureGroups* usa los índices del grupo de captura y la referencia del grupo de captura con nombre para capturar los valores coincidentes.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(?P<first>\w)(?P<middle>\w+)(?P<last>\w)", dynamic(['first',2,'last']), Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8", "2b8be2", "d"], ["d", "FA", "7"], ["4", "BD", "1"], ["8", "F6", "3"], ["2", "4ad26d3144", "9"]]|

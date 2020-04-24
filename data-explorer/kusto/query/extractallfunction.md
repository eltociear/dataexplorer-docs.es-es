---
title: extract_all() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe extract_all() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f7623a064e272f8b96ca25cc6af47318e26dfb93
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515426"
---
# <a name="extract_all"></a>extract_all()

Obtener todas las coincidencias de una [expresión regular](./re2.md) de una cadena de texto.

Opcionalmente, se puede recuperar un subconjunto de grupos coincidentes.

```kusto
print extract_all(@"(\d+)", "a set of numbers: 123, 567 and 789") == dynamic(["123", "567", "789"])
```

**Sintaxis**

`extract_all(`*regex* `,` [*captureGroups*`,`] *texto*`)`

**Argumentos**

* *regex*: Una [expresión regular](./re2.md). La expresión regular debe tener al menos un grupo de captura y menor o igual que 16 grupos de captura.
* *captureGroups*: (opcional). Constante de matriz dinámica que indica el grupo de captura que se ha extraído. Los valores válidos son de 1 al número de grupos de captura en la expresión regular. También se permiten grupos de captura con nombre (consulte la sección de ejemplos).
* *texto*: `string` A para buscar.

**Devuelve**

Si *regex* encuentra una coincidencia en el *texto:* devuelve una matriz dinámica que incluye todas las coincidencias con los grupos de captura indicados *captureGroups* (o todos los grupos de captura en el *regex*).
Si number of *captureGroups* es 1: la matriz devuelta tiene una única dimensión de valores coincidentes.
Si el número de *captureGroups* es mayor que 1: la matriz devuelta es una colección bidimensional de coincidencias multivalor por selección *captureGroups* (o todos los grupos de captura presentes en el *regex* si se omite *captureGroups)* 

Si no hay coincidencia: `null`. 

**Ejemplos**

### <a name="extracting-single-capture-group"></a>Extraer un solo grupo de captura
El ejemplo siguiente devuelve la representación de hexbytes (dos dígitos hexadecimales) del GUID.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"([\da-f]{2})", Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|["82","b8","be","2d",df","a7","4b","d1","8f","63","24","ad","26","d3","14","49"]|

### <a name="extracting-several-capture-groups"></a>Extraer varios grupos de captura 
El siguiente ejemplo utiliza una expresión regular con 3 grupos de captura para dividir cada parte GUID en la primera letra, la última letra y lo que sea en el medio.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[[8","2b8be2","d"],["d","fa","7"],["4","bd","1"],["8","f6","3"],["2","4ad26d3144","9"]]|

### <a name="extracting-subset-of-capture-groups"></a>Extraer subconjunto de grupos de captura

El siguiente ejemplo muestra cómo seleccionar un subconjunto de grupos de captura: en este caso, la expresión regular coincide con la primera letra, la última letra y el resto, mientras que el parámetro *captureGroups* se utiliza para seleccionar solo la primera y la última parte. 

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", dynamic([1,3]), Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[[8","d"],["d","7"],["4","1"],["8","3"],["2","9"]]|


### <a name="using-named-capture-groups"></a>Uso de grupos de captura con nombre

Puede utilizar grupos de captura con nombre de RE2 en extract_all(). En el ejemplo siguiente: *captureGroups* utiliza índices de grupo de captura y referencia de grupo de captura con nombre para capturar valores coincidentes.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(?P<first>\w)(?P<middle>\w+)(?P<last>\w)", dynamic(['first',2,'last']), Id) 
```

|Identificador|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[[8","2b8be2","d"],["d","fa","7"],["4","bd","1"],["8","f6","3"],["2","4ad26d3144","9"]]|
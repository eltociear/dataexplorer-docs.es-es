---
title: 'extract_all (): Explorador de datos de Azure'
description: En este artículo se describe extract_all () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 55168d381ab69bf0d29e8560714e13cb635fd688
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780531"
---
# <a name="extract_all"></a>extract_all()

Obtiene todas las coincidencias de una [expresión regular](./re2.md) de una cadena de texto.
Opcionalmente, recupere un subconjunto de grupos coincidentes.

```kusto
print extract_all(@"(\d+)", "a set of numbers: 123, 567 and 789") // results with the dynamic array ["123", "567", "789"]
```

**Sintaxis**

`extract_all(`*Regex* `,` [*captureGroups* `,` ] *texto* de`)`

**Argumentos**

|Argumento        |Descripción                                  |Obligatorio u opcional  |
|----------------|---------------------------------------------|----------------------|
|regex           | [Expresión regular](./re2.md). La expresión debe tener al menos un grupo de captura, y menor o igual que 16 grupos de captura                                                         |Requerido              |
|captureGroups   |Constante de matriz dinámica que indica el grupo de captura que se va a extraer. Los valores válidos van de 1 al número de grupos de captura en la expresión regular. También se permiten grupos de captura con nombre (vea [ejemplos](#examples))|Opcionales         |
|text            |`string`Que se va a buscar.                         |Requerido              |

**Devuelve**

* Si *Regex* encuentra una coincidencia en el *texto*: devuelve una matriz dinámica que incluye todas las coincidencias en los grupos de capturas indicados *captureGroups*, o en todos los grupos de captura de la *expresión regular*.
* Si el número de *captureGroups* es 1: la matriz devuelta tiene una única dimensión de valores coincidentes.
* Si el número de *captureGroups* es mayor que 1: la matriz devuelta es una colección bidimensional de coincidencias de varios valores por selección de *captureGroups* , o todos los grupos de captura presentes en el *Regex* si se omite *captureGroups* .
* Si no hay ninguna coincidencia: `null` .

## <a name="examples"></a>Ejemplos

### <a name="extract-a-single-capture-group"></a>Extraer un solo grupo de captura

Devuelve la representación de bytes hexadecimales (dos dígitos hexadecimales) del GUID.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"([\da-f]{2})", Id) 
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|["82", "B8", "es", "2D", "DF", "A7", "4B", "D1", "8F", "63", "24", "ad", "26", "D3", "14", "49"]|

### <a name="extract-several-capture-groups"></a>Extraer varios grupos de capturas 

Usa una expresión regular con tres grupos de captura para dividir cada parte del GUID en la primera letra, la última letra y cualquier elemento que se encuentra en el centro.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", Id)
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8", "2b8be2", "d"], ["d", "FA", "7"], ["4", "BD", "1"], ["8", "F6", "3"], ["2", "4ad26d3144", "9"]]|

### <a name="extract-a-subset-of-capture-groups"></a>Extraer un subconjunto de grupos de capturas

Muestra cómo seleccionar un subconjunto de grupos de captura. La expresión regular coincide con la primera letra, la última letra y todo el resto. El parámetro *captureGroups* se usa para seleccionar solo la primera y la última parte.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(\w)(\w+)(\w)", dynamic([1,3]), Id) 
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8", "d"], ["d", "7"], ["4", "1"], ["8", "3"], ["2", "9"]]|

### <a name="using-named-capture-groups"></a>Usar grupos de captura con nombre

Puede usar grupos de captura con nombre de RE2 en extract_all ().
*CaptureGroups* usa ambos índices de grupo de captura y la referencia de grupo de captura con nombre para capturar los valores coincidentes.

```kusto
print Id="82b8be2d-dfa7-4bd1-8f63-24ad26d31449"
| extend guid_bytes = extract_all(@"(?P<first>\w)(?P<middle>\w+)(?P<last>\w)", dynamic(['first',2,'last']), Id) 
```

|ID|guid_bytes|
|---|---|
|82b8be2d-dfa7-4bd1-8f63-24ad26d31449|[["8", "2b8be2", "d"], ["d", "FA", "7"], ["4", "BD", "1"], ["8", "F6", "3"], ["2", "4ad26d3144", "9"]]|

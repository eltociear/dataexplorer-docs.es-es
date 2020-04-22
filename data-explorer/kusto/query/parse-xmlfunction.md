---
title: parse_xml() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_xml() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ace60d0f9652d75b0f55cce4b1b622af20b42dc1
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663691"
---
# <a name="parse_xml"></a>parse_xml()

Interpreta `string` a como un valor XML, convierte el valor en `dynamic`JSON y devuelve el valor como .

**Sintaxis**

`parse_xml(`*Xml*`)`

**Argumentos**

* *xml*: Una `string`expresión de tipo , que representa un valor con formato XML.

**Devuelve**

Objeto de tipo [dynamic](./scalar-data-types/dynamic.md) determinado por el valor de *xml*, o null, si el formato XML no es válido.

La conversión del XML a JSON se realiza mediante la biblioteca [xml2json.](https://github.com/Cheedoong/xml2json)

La conversión se realiza de la siguiente manera:

XML                                |JSON                                            |Acceso
-----------------------------------|------------------------------------------------|--------------         
`<e/>`                             | • "e": nulo ?                                  | O.e
`<e>text</e>`                      | • "e": "texto"                                | O.e
`<e name="value" />`               | • "e":"@name": "valor"                     | o.e["@name"]
`<e name="value">text</e>`         | • "e":@name": "valor", "#text": "texto" | o.e["@name"] o.e["#text"]
`<e> <a>text</a> <b>text</b> </e>` | • "e": "a": "texto", "b": "texto" ?          | o.e.a o.e.b
`<e> <a>text</a> <a>text</a> </e>` | • "e": "a": ["texto", "texto"]             | o.e.a[0] o.e.a[1]
`<e> text <a>text</a> </e>`        | • "e": "#text": "texto", "a": "texto"      | 1'e.e["#text"] o.e.a

**Notas**

* La longitud `string` máxima `parse_xml` de entrada para es de 128 KB. La interpretación de cadenas más largas dará como resultado un objeto nulo 
* Solo se traducirán nodos de elemento, atributos y nodos de texto. Todo lo demás se omitirá
 
**Ejemplo**

En el ejemplo siguiente, cuando `context_custom_metrics` es un elemento `string` similar a este:
<!--check this code formatting-->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<duration>
    <value>118.0</value>
    <count>5.0</count>
    <min>100.0</min>
    <max>150.0</max>
    <stdDev>0.0</stdDev>
    <sampledValue>118.0</sampledValue>
    <sum>118.0</sum>
</duration>
```

a continuación, el siguiente fragmento de CSL traduce el XML al siguiente JSON:

```json
{
    "duration": {
        "value": 118.0,
        "count": 5.0,
        "min": 100.0,
        "max": 150.0,
        "stdDev": 0.0,
        "sampledValue": 118.0,
        "sum": 118.0
    }
}
```

y recupera el valor `duration` de la ranura en el objeto, `duration.value` y `duration.min` `118.0` de `110.0`eso recupera dos ranuras, y ( y , respectivamente).

```kusto
T
| extend d=parse_xml(context_custom_metrics) 
| extend duration_value=d.duration.value, duration_min=d["duration"]["min"]
```
---
title: Nombres de entidad - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen los nombres de entidad en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2019
ms.openlocfilehash: 2fecbd538a53d8eb02e55b0bd1def2186467b019
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509425"
---
# <a name="entity-names"></a>Nombres de entidad

Se nombran las entidades Kusto (bases de datos, tablas, columnas y funciones almacenadas; los clústeres son una excepción). El nombre de una entidad identifica la entidad y se garantiza que es único en el ámbito de su contenedor dado su tipo.
(Por ejemplo, dos tablas de la misma base de datos no pueden tener el mismo nombre, pero una tabla y una base de datos pueden tener el mismo nombre porque no están en el mismo ámbito y una tabla y una función almacenada pueden tener el mismo nombre porque no son del mismo tipo de entidad.)

Los nombres de entidad **distinguen mayúsculas de minúsculas** para fines de resolución (por ejemplo, no se puede hacer referencia a una tabla denominada `ThisTable` ). `thisTABLE`

Los nombres de entidad son un ejemplo de **identificadores.** Otros identificadores incluyen los nombres de los parámetros de las funciones y enlazar un nombre a través de una [instrucción let.](../letstatement.md)

## <a name="entity-pretty-names"></a>Nombres bonitos de la entidad

Algunas entidades (como bases de datos) pueden tener, además de su nombre de entidad, un **nombre bonito.** Los nombres bonitos se pueden usar para hacer referencia a la entidad en consultas (como nombres de entidad), pero, a diferencia de los nombres de entidad, los nombres bonitos no son necesariamente únicos en el contexto de su contenedor. Cuando un contenedor tiene varias entidades con el mismo nombre bonito, el nombre bonito no se puede usar para hacer referencia a la entidad.

Los nombres bonitos permiten que las aplicaciones de nivel intermedio se asignen automáticamente a nombres de entidad (como UUID) a nombres que son legibles por humanos para fines de visualización y referencia.

## <a name="identifier-naming-rules"></a>Reglas de nomenclatura de identificadores

<!-- TODO: This section should be reviewed and moved to its own page -->

Los identificadores se utilizan para asignar nombres a varias entidades (entidades o de otro modo).
Los nombres de identificador válidos siguen estas reglas:
* Tienen entre 1 y 1024 caracteres de largo.
* Pueden contener letras, dígitos,`_`guiones bajos`.`( ), espacios, puntos ( ) y guiones (`-`).
  * Los identificadores que constan solo de letras, dígitos y guiones bajos no requieren comillas cuando se hace referencia al identificador.
  * Los identificadores que contienen por fin uno de (espacios, puntos o guiones) requieren comillas (ver más abajo).
* Distinguen mayúsculas de minúsculas.

## <a name="identifier-quoting"></a>Cita identificadora

Los identificadores que son idénticos a algunas palabras clave de lenguaje de consulta, o que tienen uno de los caracteres especiales indicados anteriormente, requieren comillas cuando una consulta hace referencia a ellos directamente:

|Texto de consulta         |Comentarios                          |
|-------------------|----------------------------------|
| `entity`          |Los nombres`entity`de entidad ( ) que no incluyen caracteres especiales o se asignan a alguna palabra clave de idioma no requieren comillas|
|`['entity-name']`  |Los nombres de entidad que `-`incluyan caracteres `['` `']` especiales `["` (aquí: ) deben ser citados usando y o usando y`"]`|
|`["where"]`        |Los nombres de entidad que son palabras `['` `']` clave `["` de idioma deben ser citados`"]`|

## <a name="naming-your-entities-to-avoid-collisions-with-kusto-language-keywords"></a>Nombrar sus entidades para evitar colisiones con palabras clave de lenguaje Kusto

Como el lenguaje de consulta Kusto incluye una serie de palabras clave que tienen las mismas reglas de nomenclatura que los identificadores, es posible tener nombres de entidad que en realidad son palabras clave, pero luego hacer referencia a estos nombres se vuelve difícil (uno debe citarlos).

Como alternativa, es posible que desee elegir nombres de entidad que se garantiza que nunca "colisionen" con una palabra clave Kusto. Se realizan las siguientes garantías:

1. El lenguaje de consulta Kusto no definirá una`A` `Z`palabra clave que comience con una letra mayúscula ( to ).
2. El lenguaje de consulta Kusto no definirá una`_`palabra clave que comience con un solo carácter de subrayado ( ).us
3. El lenguaje de consulta Kusto no definirá una`_`palabra clave que termine con un solo carácter de subrayado ( ).

El lenguaje de consulta Kusto reserva todos los identificadores que`__`comienzan o terminan con una secuencia de dos caracteres de subrayado ( ); los usuarios no pueden definir dichos nombres para su propio uso.









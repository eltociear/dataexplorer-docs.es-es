---
title: El tipo de datos de cadena- Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el tipo de datos de cadena en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 8c040045ba7146cf56487ca3a8729372084d3bf2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509629"
---
# <a name="the-string-data-type"></a>El tipo de datos de cadena

El `string` tipo de datos representa una cadena Unicode. (Las cadenas Kusto están codificadas en UTF-8 y, de forma predeterminada, están limitadas a 1 MB.)

## <a name="string-literals"></a>Literales de cadena

Hay varias maneras de codificar `string` literales del tipo de datos:

* Encerrando la cadena entre comillas`"`dobles ( ):`"This is a string literal. Single quote characters (') do not require escaping. Double quote characters (\") are escaped by a backslash (\\)"`
* Encerrando la cadena entre comillas`'`simples ( ):`'Another string literal. Single quote characters (\') require escaping by a backslash (\\). Double quote characters (") do not require escaping.'`

En las dos representaciones anteriores, el carácter de barra diagonal inversa (`\`) indica el escape.
Se utiliza para escapar de los caracteres`\t`de comillas`\n`envolventes,`\\`caracteres de tabulación ( ), caracteres de nueva línea ( ) y sí mismo ( ).

También se admiten literales de cadena literales. En esta forma, el`\`carácter de barra diagonal inversa ( ) se representa a sí mismo, no como un carácter de escape:

* Encerrado entre comillas`"`dobles ( ):`@"This is a verbatim string literal that ends with a backslash\"`
* Encerrado entre comillas`'`simples ( ):`@'This is a verbatim string literal that ends with a backslash\'`

Dos literales de cadena en el texto de consulta sin nada entre ellos, o separados sólo por espacios en blanco y comentarios, se concatenan automáticamente para formar un nuevo literal de cadena (hasta que no se pueda realizar dicha sustitución).
Por ejemplo, todas las `13`expresiones siguientes producen:

```kusto
print strlen("Hello"', '@"world!"); // Nothing between them

print strlen("Hello" ', ' @"world!"); // Separated by whitespace only

print strlen("Hello"
  // Comment
  ', '@"world!"); // Separated by whitespace and a comment
```

## <a name="examples"></a>Ejemplos

```kusto
// Simple string notation
print s1 = 'some string', s2 = "some other string"

// Strings that include single or double-quotes can be defined as follows
print s1 = 'string with " (double quotes)',
          s2 = "string with ' (single quotes)"

// Strings with '\' can be prefixed with '@' (as in c#)
print myPath1 = @'C:\Folder\filename.txt'

// Escaping using '\' notation
print s = '\\n.*(>|\'|=|\")[a-zA-Z0-9/+]{86}=='
```

Como se puede ver, cuando una cadena se`"`encierra entre comillas`'`dobles ( ), el carácter de comillas simples ( ) no requiere escape y viceversa. Esto facilita la cita de cadenas según el contexto.

## <a name="obfuscated-string-literals"></a>Literales de cadena ofuscados

El sistema realiza un seguimiento de las consultas y las almacena con fines de telemetría y análisis.
Por ejemplo, el texto de la consulta podría estar disponible para el propietario del clúster. Si el texto de la consulta incluye información secreta (como contraseñas), es posible que se filtre información que debe mantenerse privada. Para evitar que esto suceda, el autor de la consulta puede marcar literales de cadena específicos como literales de **cadena ofuscados.**
Estos literales en el texto de la consulta`*`se reemplazan automáticamente por un número de caracteres de estrella ( ), de modo que no están disponibles para su análisis posterior.

> [!IMPORTANT]
> Se **recomienda encarecidamente** que todos los literales de cadena que contienen información secreta se marquen como literales de cadena ofuscados.

Un literal de cadena ofuscado se puede formar tomando un `h` literal `H` de cadena "regular" y anteponiendo uno o un carácter delante de él. Por ejemplo:

```kusto
h'hello'
h@'world'
h"hello"
```

> [!NOTE]
> En muchos casos, solo una parte del literal de cadena es secreta. Es muy útil en esos casos dividir el literal en una parte no secreta y una parte secreta, a continuación, sólo marcar la parte secreta como ofuscado. Por ejemplo:

```kusto
print x="https://contoso.blob.core.windows.net/container/blob.txt?"
  h'sv=2012-02-12&se=2013-04-13T0...'
```
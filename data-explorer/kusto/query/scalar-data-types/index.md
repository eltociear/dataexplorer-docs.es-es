---
title: 'Tipos de datos escalares: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describen los tipos de datos escalares de Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: 3ef87217beee62fe4cecf7ee95dfe8daba49af7a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490249"
---
# <a name="scalar-data-types"></a>Tipos de datos escalares

Cada valor de datos (como el valor de una expresión o el parámetro de una función) tiene un **tipo de datos**. Los tipos de datos se clasifican en **tipos de datos escalares** (uno de los tipos predefinidos integrados que se enumeran a continuación) o **registros definidos por el usuario** (una secuencia ordenada de pares de nombre y tipo de datos escalar, como el tipo de datos de una fila de una tabla).

Kusto proporciona un conjunto de tipos de datos de sistema que define todos los tipos de datos que se pueden usar con Kusto.

> [!NOTE]
> En Kusto no se admiten los tipos de datos definidos por el usuario.

En la tabla siguiente se enumeran los tipos de datos que admite Kusto, junto con los alias adicionales que se pueden usar para hacer referencia a ellos y un tipo de .NET Framework equivalente.

| Tipo       | Nombres adicionales   | Tipo equivalente de .NET              | gettype()   |Tipo de almacenamiento (nombre interno)|
| ---------- | -------------------- | --------------------------------- | ----------- |----------------------------|
| `bool`     | `boolean`            | `System.Boolean`                  | `int8`      |`I8`                        |
| `datetime` | `date`               | `System.DateTime`                 | `datetime`  |`DateTime`                  |
| `dynamic`  |                      | `System.Object`                   | `array` o `dictionary`, o cualquiera de los otros valores |`Dynamic`|
| `guid`     | `uuid`, `uniqueid`   | `System.Guid`                     | `guid`      |`UniqueId`                  |
| `int`      |                      | `System.Int32`                    | `int`       |`I32`                       |
| `long`     |                      | `System.Int64`                    | `long`      |`I64`                       |
| `real`     | `double`             | `System.Double`                   | `real`      |`R64`                       |
| `string`   |                      | `System.String`                   | `string`    |`StringBuffer`              |
| `timespan` | `time`               | `System.TimeSpan`                 | `timespan`  |`TimeSpan`                  |
| `decimal`  |                      | `System.Data.SqlTypes.SqlDecimal` | `decimal`   | `Decimal`                  |

Todos los tipos de datos incluyen un valor especial "NULL", que indica que faltan datos o que los datos no coinciden. Por ejemplo, si se intenta ingerir la cadena `"abc"` en una columna `int`, se genera este valor.
Este valor no se puede materializar explícitamente, pero mediante la función `isnull()` se puede detectar si el resultado de la evaluación de una expresión es este valor.

> [!WARNING]
> En el momento en que se redactó este documento, la compatibilidad con el tipo `guid` no es completa. Es muy recomendable que los equipos usen valores del tipo `string` en su lugar.


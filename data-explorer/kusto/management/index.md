---
title: 'Introducción a la administración (comandos de control): Azure Data Explorer | Microsoft Docs'
description: En este artículo se proporciona información general acerca de la administración (comandos de control) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1277de1be577de7f9f6d4adf1b74460eac4a8c42
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490351"
---
# <a name="management-control-commands-overview"></a>Introducción a la administración (comandos de control)

En esta sección se describen los comandos de control que se usan para administrar Kusto.
Los comandos de control son solicitudes que se realizan al servicio para recuperar información que no son necesariamente los datos de las tablas de base de datos, para modificar el estado del servicio, etc.

## <a name="differentiating-control-commands-from-queries"></a>Diferencias entre los comandos de control y las consultas

Kusto usa tres mecanismos para diferenciar las consultas y los comandos de control: en el nivel de lenguaje, en el nivel de protocolo y en el nivel de API. Esto se hace por motivos de seguridad.

En el nivel de lenguaje, el primer carácter del texto de una solicitud determina si la solicitud es un comando de control o una consulta. Los comandos de control deben comenzar por un punto (`.`) y ninguna consulta puede comenzar por ese carácter.

En el nivel de protocolo, se usan puntos de conexión HTTP/HTTPS diferentes para los comandos de control y para las consultas.

En el nivel de API, se usan funciones diferentes para enviar comandos de control y consultas.

## <a name="combining-queries-and-control-commands"></a>Combinación de consultas y comandos de control

Los comandos de control pueden hacer referencia a las consultas (pero no al contrario) o a otros comandos de control.
Se admiten varios escenarios:

1. **AdminThenQuery**: se ejecuta un comando de control y su resultado (representado en forma de tabla de datos temporal) sirve como entrada para una consulta.
2. **AdminFromQuery**: se ejecuta una consulta o un comando de administración `.show`, y su resultado (representado en forma de tabla de datos temporal) sirve como entrada para un comando de control.

Tenga en cuenta que, en todos los casos, técnicamente la combinación completa es un comando de control, no una consulta, por lo que el texto de la solicitud debe comenzar por un carácter de punto (`.`) y la solicitud se debe enviar al punto de conexión de administración del servicio.

Tenga también en cuenta que las [instrucciones de consulta](../query/statements.md) aparecen en la parte de consulta del texto (no pueden preceder al propio comando).

**AdminThenQuery** se indica de una de estas dos maneras:

1. Mediante el uso de un carácter de barra vertical (`|`); por lo tanto, la consulta trata los resultados del comando de control como si fuera cualquier otro operador de consulta que genera datos.
2. Mediante el uso de un carácter de punto y coma (`;`) que luego introduce los resultados del comando de control en un símbolo especial denominado `$command_results` y que posteriormente se puede usar en la consulta tantas veces como se desee.

Por ejemplo:

```kusto
// 1. Using pipe: Count how many tables are in the database-in-scope:
.show tables
| count

// 2. Using semicolon: Count how many tables are in the database-in-scope:
.show tables;
$command_results
| count

// 3. Using semicolon, and including a let statement:
.show tables;
let useless=(n:string){strcat(n,'-','useless')};
$command_results | extend LastColumn=useless(TableName)
```

**AdminFromQuery** se indica mediante la combinación de caracteres `<|`. En el siguiente ejemplo, primero se ejecuta una consulta que genera una tabla con una sola columna (llamada `str` del tipo `string`) y una sola fila, y se escribe como el nombre de la tabla `MyTable` en la base de datos en contexto:

```kusto
.set MyTable <|
let text="Hello, World!";
print str=Text
```

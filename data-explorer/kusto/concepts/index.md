---
title: 'Introducción a Kusto: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describe cómo empezar a usar Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5b66dd59dd17f1671c68e63e35ee62f56e6ec8fe
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610348"
---
# <a name="getting-started-with-kusto"></a>Introducción a Kusto

Kusto es un servicio para almacenar y ejecutar análisis interactivos sobre macrodatos.

Se basa en sistemas de administración de bases de datos relacionales, admite entidades como bases de datos, tablas y columnas, y además proporciona operadores de consulta de análisis complejos (como columnas calculadas, búsqueda y filtrado por filas, agrupar por agregados o uniones).

Kusto ofrece una ingesta de datos y un rendimiento de consultas excelentes, y para ello "sacrifica" la capacidad de realizar actualizaciones en contexto de filas individuales y restricciones y transacciones entre tablas. Por tanto, en lugar de reemplazar los sistemas de administración de bases de datos relacionales tradicionales en escenarios como OLTP y el almacenamiento de datos, los complementa.

Al ser un servicio de macrodatos, Kusto gestiona de datos estructurados, semiestructurados (tipos anidados similares a JSON) y no estructurados (texto libre) igualmente de bien.

## <a name="interacting-with-kusto"></a>Interacción con Kusto

La forma principal en que los usuarios interactúen con Kusto es mediante una de las numerosas [herramientas de cliente](../tools/index.md) disponibles para Kusto. Aunque se admiten [las consultas SQL](../api/tds/t-sql.md) a Kusto, el medio principal de interacción con Kusto es el [lenguaje de consulta de Kusto ](../query/index.md) para enviar consultas de datos y mediante el uso de [comandos de control](../management/index.md) para administrar entidades Kusto, detectar metadatos, etc. Tanto las consultas como los comandos de control son básicamente "programas" de texto corto.

## <a name="queries"></a>Consultas

Una consulta de Kusto es una solicitud de solo lectura para procesar datos de Kusto y devolver los resultados de este procesamiento, sin modificar los datos o metadatos de Kusto. Las consultas de Kusto pueden [usar el](../api/tds/t-sql.md)lenguaje SQL o el [lenguaje de consulta de Kusto](../query/index.md).
Como ejemplo de este último, la consulta siguiente cuenta el número de filas de la `Logs`tabla que tienen el valor de la`Level` columna igual a la cadena`Critical`:

```kusto
Logs
| where Level == "Critical"
| count
```

Las consultas no pueden empezar con el carácter de punto (`.`) o el carácter de hash (`#`).

## <a name="control-commands"></a>Comandos de control

Los comandos de control son solicitudes a Kusto para procesar y, potencialmente, modificar datos o metadatos. Por ejemplo, el siguiente comando de control crea una nueva tabla Kusto con dos columnas, `Level` y `Text`:

```kusto
.create table Logs (Level:string, Text:string)
```

Los comandos de control tienen su propia sintaxis (que no forma parte de la sintaxis del lenguaje de consulta de Kusto, aunque ambas comparten muchos conceptos). En concreto, los comandos de control se distinguen de las consultas en que el primer carácter del texto del comando es un punto (`.`) (que no puede iniciar una consulta).
Esta distinción evita muchos tipos de ataques de seguridad, simplemente porque esto evita la incrustación de comandos de control dentro de las consultas.

No todos los comandos de control modifican los datos o metadatos de Kusto. Una clase grande de comandos, los que comienzan por `.show`, se usan para mostrar metadatos o datos sobre Kusto. Por ejemplo, el comando `.show tables` devuelve una lista de todas las tablas de la base de datos actual.

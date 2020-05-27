---
title: Introducción a Kusto
description: En este artículo se describen los primeros pasos con Kusto.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0a56878c8b79e651afcd1b8ce18a3220dc304211
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863394"
---
# <a name="getting-started-with-kusto"></a>Introducción a Kusto

Azure Data Explorer es un servicio para almacenar y ejecutar análisis interactivos en macrodatos.

Se basa en sistemas de administración de bases de datos relacionales que admiten entidades como bases de datos, tablas y columnas. Las consultas analíticas complejas se realizan mediante el lenguaje de consulta de Kusto. Algunos operadores de consulta incluyen columnas calculadas, búsquedas y filtros, o filas, agregados de agrupar por y combinaciones.

El servicio ofrece una ingesta de datos y un rendimiento de consultas excelentes, y para ello "sacrifica" la capacidad de realizar actualizaciones en contexto de filas individuales y restricciones y transacciones entre tablas. Por tanto, en lugar de reemplazar los sistemas de administración de bases de datos relacionales tradicionales en escenarios como OLTP y el almacenamiento de datos, los complementa.

Los datos estructurados, semiestructurados (tipos anidados similares a JSON) y no estructurados (texto libre) se gestionan igual de bien.

## <a name="interacting-with-azure-data-explorer"></a>Interacción con Azure Data Explorer

La forma principal en que los usuarios interactúan con Kusto es mediante una de las numerosas [herramientas de cliente](../tools/index.md) disponibles. Aunque se admiten [consultas SQL](../api/tds/t-sql.md), el principal medio de interacción es el [lenguaje de consulta de Kusto ](../query/index.md) para enviar consultas de datos y mediante el uso de [comandos de control](../management/index.md) para administrar entidades, detectar metadatos, etc. Tanto las consultas como los comandos de control son básicamente "programas" de texto corto.

## <a name="kusto-queries"></a>Consultas de Kusto

Una consulta es una solicitud de solo lectura para procesar datos y devolver los resultados de este procesamiento, sin modificar los datos o metadatos. Las consultas de Kusto pueden [usar el](../api/tds/t-sql.md)lenguaje SQL o el [lenguaje de consulta de Kusto](../query/index.md). Como ejemplo de este último, la consulta siguiente cuenta el número de filas de la `Logs`tabla que tienen el valor de la`Level` columna igual a la cadena`Critical`:

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

No todos los comandos de control modifican los datos o metadatos. Una clase grande de comandos, los que comienzan por `.show`, se usan para mostrar metadatos o datos. Por ejemplo, el comando `.show tables` devuelve una lista de todas las tablas de la base de datos actual.

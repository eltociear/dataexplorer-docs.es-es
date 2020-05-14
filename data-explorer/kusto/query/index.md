---
title: Introducción a Azure Data Explorer
description: En este artículo se proporciona información general acerca de Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/07/2019
ms.openlocfilehash: 7f36e5239ba183b87ab04349b873a0182363def7
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226489"
---
# <a name="overview"></a>Información general

Una consulta de Kusto es una solicitud de solo lectura para procesar los datos y devolver resultados.
La solicitud se indica con un texto sin formato, mediante un modelo de flujo de datos diseñado para simplificar la lectura, creación y automatización de la sintaxis. La consulta usa entidades de esquema organizadas es una jerarquía similar a las de las bases de datos, tablas y columnas de SQL.

La consulta consta de una secuencia de instrucciones de consulta delimitadas por un punto y coma (`;`) y al menos una de ellas es una [instrucción de expresiones tabulares](tabularexpressionstatements.md), que es una instrucción que genera datos organizados en una malla tabular de columnas y filas. Las instrucciones de expresión tabular de la consulta generan los resultados de la consulta.

La sintaxis de la instrucción de expresiones tabulares tiene un flujo de datos tabulares desde un operador de consultas tabulares a otro, empezando por el origen de datos (por ejemplo, una tabla en una base de datos o un operador que genera datos) y, después, pasando por un conjunto de operadores de transformación de datos que se unen mediante el delimitador de barra vertical (`|`).

Por ejemplo, la siguiente consulta de Kusto tiene una sola instrucción, que es una instrucción de expresiones tabulares. La instrucción se inicia con una referencia a una tabla denominada `StormEvents` (aquí la base de datos que hospeda esta tabla es implícita y forma parte de la información de conexión). Después, los datos (filas) de esa tabla se filtran por el valor de la columna `StartTime` y, luego, según el valor de la columna `State`. Después, la consulta devuelve el recuento de filas que "sobreviven".

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where StartTime >= datetime(2007-11-01) and StartTime < datetime(2007-12-01)
| where State == "FLORIDA"  
| count 
```

Para ejecutar esta consulta, [haga clic aquí](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAAwsuyS/KdS1LzSspVuDlqlEoz0gtSlUILkksKgnJzE1VsLNVSEksSS0BsjWMDAzMdQ0NdQ0MNRUS81KQVNmgKzICKUIxryRVwdZWQcnNxz/I08VRSQFsW3J+aV6JAgAwMx4+hAAAAA==).
En este caso, el resultado será:

|Count|
|-----|
|   23|

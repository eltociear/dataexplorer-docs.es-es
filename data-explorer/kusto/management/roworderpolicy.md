---
title: 'Directiva de orden de fila: Explorador de azure Data Explorer ( Row Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de orden de fila en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 4dca1a4083a6141eb94d9bf3cf69f7134ebfca04
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520220"
---
# <a name="row-order-policy"></a>Directiva de orden de filas

La directiva de orden de filas es un conjunto de políticas opcional en tablas, que proporciona una "sugerencia" a Kusto en el orden deseado de filas en una partición de datos.

El propósito principal de la directiva no es mejorar la compresión (aunque es un posible efecto secundario), sino mejorar el rendimiento de las consultas que se sabe que se reducen a un pequeño subconjunto de valores en las columnas ordenadas.

La aplicación de la política es apropiada cuando:
* La mayoría de las consultas filtran valores específicos de una columna de gran dimensión específica (como un "identificador de aplicación" o un "identificador de inquilino")
* Es poco probable que los datos ingeridos en la tabla se ordenen previamente según esta columna.

Aunque no hay límites codificados de forma rígida establecidos en la cantidad de columnas (claves de ordenación) que se pueden definir como parte de la directiva, cada columna adicional agrega cierta sobrecarga al proceso de ingesta y, a medida que se agregan más columnas, el retorno efectivo disminuye.

> [!NOTE]
> Una vez que la directiva se aplica a una tabla, afectará a los datos ingeridos a partir de ese momento.

Los comandos de control para administrar las directivas de orden de fila se pueden encontrar [aquí](../management/roworder-policy.md)
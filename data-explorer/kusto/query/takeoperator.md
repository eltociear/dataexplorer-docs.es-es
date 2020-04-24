---
title: 'operador de toma: Explorador de datos de Azure. Microsoft Docs'
description: En este artículo se describe el operador take en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0c8f724e139e13bf9ece00d5af09f2a3cf25b03a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506603"
---
# <a name="take-operator"></a>Operador take

Vuelva hasta el número especificado de filas.

```kusto
T | take 5
```

No hay ninguna garantía de qué registros se devuelven, a menos que se ordenen los datos de origen.

**Sintaxis**

`take`*NumberOfRows* 
 `limit` *NumberOfRows*

(`take` `limit` y son sinónimos.)

**Notas**

`take`es una forma sencilla, rápida y eficiente de ver una pequeña muestra de registros al examinar datos de forma interactiva, pero tenga en cuenta que no garantiza ninguna coherencia en sus resultados al ejecutar se ejecutan varias veces, incluso si el conjunto de datos no ha cambiado.

Incluso es el número de filas devueltas por la `take` consulta no está explícitamente limitado por la consulta (no se utiliza ningún operador), Kusto limita ese número de forma predeterminada.
Consulte Límites de consulta de [Kusto](../concepts/querylimits.md) para obtener más información.

Consulte: operador de [ordenación](sortoperator.md)
operador[top](topoperator.md)
[anidado operador](topnestedoperator.md)

## <a name="does-kusto-support-paging-of-query-results"></a>¿Kusto admite la paginación de los resultados de la consulta?

Kusto no proporciona un mecanismo de paginación integrado.

Kusto es un servicio complejo que optimiza continuamente los datos que almacena para proporcionar un excelente rendimiento de las consultas en grandes conjuntos de datos. Aunque la paginación es un mecanismo útil para clientes sin estado con recursos limitados, desplaza la carga al servicio back-end que tiene que realizar un seguimiento de la información de estado del cliente. Posteriormente, el rendimiento y la escalabilidad del servicio back-end se limitan gravemente.

Para la compatibilidad con paginación, implemente una de las siguientes características:

* Exportar el resultado de una consulta a un almacenamiento externo y paginar a través de los datos generados.

* Escribir una aplicación de nivel intermedio que proporciona una API de paginación con estado almacenando en caché los resultados de una consulta de Kusto.
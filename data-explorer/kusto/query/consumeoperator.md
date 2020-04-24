---
title: 'operador de consumo: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el operador consume en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 65c2f2befc074042131b5c0d705fa942a1622035
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517126"
---
# <a name="consume-operator"></a>Operador consume

Consume la secuencia de datos tabulares entregada al operador. 

El `consume` operador se utiliza principalmente para desencadenar el efecto secundario de la consulta sin devolver realmente los resultados al autor de la llamada.

```kusto
T | consume
```

**Sintaxis**

`consume`[`decodeblocks` `=` *DecodeBlocks*]

**Argumentos**

* *DecodeBlocks*: Un valor booleano constante. Si se `true`establece en , `perftrace` o `true`si `consume` la propiedad request está establecida en , el operador no solo enumerará los registros en su entrada, sino que realmente forzará cada valor de esos registros a descomprimirse y descodificarse.

El `consume` operador se puede utilizar para estimar el costo de una consulta sin devolver realmente los resultados al cliente.
(La estimación no es exacta por una `consume` variedad de razones; `T | consume` por ejemplo, se calcula distributivamente, por lo que no transmitirá los datos de la tabla entre los nodos del clúster.)

<!--
* *WithStats*: A constant Boolean value. If set to `true` (or if the global
  property `perftrace` is set), the operator will return a single
  row with a single column called `Stats` of type `dynamic` holding the statistics
  of the data source fed to the `consume` operator.
-->
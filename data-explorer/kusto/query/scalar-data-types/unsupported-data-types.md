---
title: 'Tipos de datos escalares no admitidos: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los tipos de datos escalares no admitidos en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: 76a548abdf05cec57e4d0558e5d98ab0f7cbdc3e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509595"
---
# <a name="unsupported-scalar-data-types"></a>Tipos de datos escalares no admitidos

Todos los tipos de **datos escalares** no documentados se consideran no admitidos en Kusto.

Entre los tipos no admitidos se encuentran los siguientes. Algunos tipos eran compatibles anteriormente:

| Tipo       | Nombres adicionales   | Tipo equivalente de .NET              | Tipo de almacenamiento (nombre interno)|
| ---------- | -------------------- | --------------------------------- | ----------------------------|
| `float`    |                      | `System.Single`                   | `R32`                       |
| `int16`    |                      | `System.Int16`                    | `I16`                       |
| `uint16`   |                      | `System.UInt16`                   | `UI16`                      |
| `uint32`   | `uint`               | `System.UInt32`                   | `UI32`                      |
| `uint64`   | `ulong`              | `System.UInt64`                   | `UI64`                      |
| `uint8`    | `byte`               | `System.Byte`                     | `UI8`                       |
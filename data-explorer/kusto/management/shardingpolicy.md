---
title: 'Directiva de particionamiento de datos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la Directiva de particionamiento de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 26ccf94f8d88c6c86a6c11c20ec9cc04f8af1a42
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011506"
---
# <a name="data-sharding-policy"></a>Directiva de particionamiento de datos

La Directiva de particionamiento define si se deben sellar las [extensiones (particiones de datos)](../management/extents-overview.md) del clúster de explorador de datos de Azure.

> [!NOTE]
> La Directiva se aplica a todas las operaciones que crean nuevas extensiones, como los comandos para la [ingesta de datos](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands)y los [comandos. Merge y. Rebuild](../management/extents-commands.md#merge-extents) .

La Directiva de particionamiento de datos contiene las siguientes propiedades:

- **MaxRowCount**:
    - Recuento máximo de filas para una extensión creada por una operación de ingesta o recompilación.
    - El valor predeterminado es 750.000.
    - **No está en vigor** para [las operaciones de combinación](mergepolicy.md).
        - Si debe limitar el número de filas en las extensiones creadas por las operaciones de combinación, ajuste la `RowCountUpperBoundForMerge` propiedad en la [Directiva de combinación de extensiones](mergepolicy.md)de la entidad.
- **MaxExtentSizeInMb**:
    - Tamaño de datos comprimido máximo permitido (en megabytes) para una extensión creada por una operación de combinación.
    - Solo se aplica **a las operaciones de [combinación](mergepolicy.md) **.
    - El valor predeterminado es 1.024 (1 GB).

- **MaxOriginalSizeInMb**:
    - Tamaño de datos original máximo permitido (en megabytes) para una extensión creada por una operación de recompilación.
    - Solo se aplica **a las operaciones de [recompilación](mergepolicy.md) **.
    - El valor predeterminado es 2.048 (2 GB).

> [!WARNING]
> Póngase en contacto con el equipo de Azure Explorador de datos antes de modificar una directiva de particionamiento de datos.

Cuando se crea una base de datos, esta contiene la Directiva de particionamiento de datos predeterminada. Todas las tablas creadas en la base de datos heredan esta Directiva (a menos que la Directiva se invalide explícitamente en el nivel de tabla).

Use los [comandos de control de directivas de particionamiento](../management/sharding-policy.md)) para administrar las directivas de particionamiento de datos para las bases de datos y tablas.
 
---
title: 'Directiva de partición de datos: Explorador de azure Data Explorer ( Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de partición de datos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 6ed9cf3a1667fb57271a44dcfe7c6dd5318c4010
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520033"
---
# <a name="data-sharding-policy"></a>Política de partición de datos

La directiva de particionamiento define si y cómo se deben sellar las [extensiones (particiones](../management/extents-overview.md) de datos) del clúster de Azure Data Explorer.

> [!NOTE]
> La directiva se aplica a todas las operaciones que crean nuevas extensiones, como comandos para la [ingesta](../management/data-ingestion/index.md)de datos, y [los comandos .merge y .rebuild](../management/extents-commands.md#merge-extents)

La directiva de partición de datos contiene las siguientes propiedades:

- **MaxRowCount**:
    - Número máximo de filas para una extensión creada por una operación de ingesta o reconstrucción.
    - El valor predeterminado es 750.000.
    - **No está en vigor** para las operaciones de [combinación.](mergepolicy.md)
        - Si debe limitar el número de filas en las `RowCountUpperBoundForMerge` extensiones creadas por las operaciones de combinación, ajuste la propiedad en la directiva de combinación de [extensiones](mergepolicy.md)de la entidad.
- **MaxExtentSizeInmb**:
    - Tamaño máximo de datos comprimidos permitido (en megabytes) para una extensión creada por una operación de combinación.
    - En **efecto, solo para operaciones de [combinación.](mergepolicy.md) **
    - El valor predeterminado es 1.024 (1 GB).

- **MaxOriginalSizeInmb**:
    - Tamaño máximo de datos original permitido (en megabytes) para una extensión creada por una operación de reconstrucción.
    - En **efecto, solo para operaciones de [reconstrucción](mergepolicy.md) **.
    - El valor predeterminado es 2.048 (2 GB).

> [!WARNING]
> Consulte con el equipo de Azure Data Explorer antes de modificar una directiva de partición de datos.

Cuando se crea una base de datos, contiene la directiva de partición de datos predeterminada. Todas las tablas creadas en la base de datos heredan esta directiva (a menos que la directiva se invalide explícitamente en el nivel de tabla).

Utilice los comandos de control de directivas de [particionamiento](../management/sharding-policy.md)) para administrar las directivas de particionamiento de datos para bases de datos y tablas.
 
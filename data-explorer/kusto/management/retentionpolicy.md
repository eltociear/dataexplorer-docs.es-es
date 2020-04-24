---
title: 'Directiva de retención: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de retención en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 81e08b6e007a6e3c669e7138e1d36ae1e701d442
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520322"
---
# <a name="retention-policy"></a>Directiva de retención

La directiva de retención controla automáticamente el mecanismo mediante el cual los datos se quitan de las tablas.
Esta eliminación suele ser útil para los datos que fluyen en una tabla continuamente cuya relevancia se basa en la edad. Por ejemplo, la directiva de retención se puede usar para una tabla que contiene eventos de diagnóstico que pueden quedar sin interestar después de dos semanas.

La directiva de retención se puede configurar para una tabla específica o para una base de datos completa (en cuyo caso se aplica a todas las tablas de la base de datos que no invalidan la directiva).

La configuración de una directiva de retención es importante para los clústeres que ingingen datos continuamente, lo que limitará los costos.

Los datos que están "fuera" de la política de retención son elegibles para su eliminación. Kusto no garantiza cuándo se produce la eliminación (por lo que los datos pueden "persistir" incluso si se ha activado la directiva de retención).

La directiva de retención se establece con mayor frecuencia para limitar la edad de los datos desde la ingesta (consulte [SoftDeletePeriod](#the-policy-object), a continuación).

> [!NOTE]
> * El tiempo de eliminación es impreciso. El sistema garantiza que los datos no se eliminarán antes de que se supere el límite, pero la supresión no es inmediata después de ese punto.
> * Se puede establecer un período de eliminación temporal de 0 como parte de una directiva de retención de nivel de tabla (pero no como parte de una directiva de retención de nivel de base de datos).
>   * Cuando esto se hace, los datos ingeridos no se confirmarán en la tabla de origen, evitando la necesidad de conservar los datos.
>   * Esta configuración es útil principalmente cuando los datos se ingestan en una tabla.
>   Se utiliza una directiva de [actualización](updatepolicy.md) transaccional para transformarla y redirigir la salida a otra tabla.

## <a name="the-policy-object"></a>El objeto de política

Una directiva de retención incluye las siguientes propiedades:

* **SoftDeletePeriod**:
    * Un intervalo de tiempo para el que se garantiza que los datos se mantienen disponibles para la consulta, medido desde el momento en que se ingirieron.
    * Su valor predeterminado es `100 years`.
    * Al modificar el período de eliminación temporal de una tabla o base de datos, el nuevo valor se aplica tanto a los datos existentes como a los nuevos.
* **Capacidad de recuperación**:
    * Capacidad de recuperación de datos (habilitada/desactivada) después de eliminar los datos
    * De manera predeterminada, su valor es `enabled`.
    * Si se `enabled`establece en , los datos serán recuperables durante 14 días después de la eliminación

## <a name="control-commands"></a>Comandos de control

* Use la retención de [directivas .show](../management/retention-policy.md) para mostrar la directiva de retención actual para una base de datos o una tabla.
* Use la retención de [directivas .alter](../management/retention-policy.md) para cambiar la directiva de retención actual de una base de datos o una tabla.

## <a name="defaults"></a>Valores predeterminados

De forma predeterminada, cuando se crea una base de datos o una tabla, no tiene definida una directiva de retención.
En casos comunes, se crea la base de datos y, a continuación, inmediatamente tiene su directiva de retención establecida por su creador de acuerdo con los requisitos conocidos.
Cuando se ejecuta un [comando show](../management/retention-policy.md) para la política de retención `Policy` de `null`una base de datos o tabla que no ha tenido su conjunto de políticas, aparece como .

La directiva de retención predeterminada (con los valores predeterminados mencionados anteriormente) se puede aplicar mediante el siguiente comando:

```kusto
.alter database DatabaseName policy retention "{}"
.alter table TableName policy retention "{}"
```

Estos resultados con el siguiente objeto de política aplicado a la base de datos o tabla:

```kusto
{
  "SoftDeletePeriod": "36500.00:00:00", "Recoverability":"Enabled"
}
```

La compensación de la directiva de retención de una base de datos o tabla se puede realizar mediante el siguiente comando:

```kusto
.delete database DatabaseName policy retention
.delete table TableName policy retention
```

## <a name="examples"></a>Ejemplos

Dado que el clúster `MyDatabase`tiene `MyTable1`una `MyTable2` base de datos denominada , con tablas y`MySpecialTable`

**1. Configuración de todas las tablas en la base de datos para tener un período de eliminación temporal de 7 días y capacidad de recuperación desactivada:**

* *Opción 1 (recomendado):* establezca una directiva de retención de nivel de base de datos con un período de eliminación temporal de siete días y capacidad de recuperación deshabilitada, y compruebe que no hay directivas de nivel de tabla establecidas.

```kusto
.delete table MyTable1 policy retention        // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention        // optional, only if the table previously had its policy set
.delete table MySpecialTable policy retention  // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
```

* *Opción 2:* Para cada tabla, establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de siete días y capacidad de recuperación deshabilitada.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 7d recoverability = disabled
```

**2. Configuración `MyTable1` `MyTable2` de tablas, para tener un período de eliminación `MySpecialTable` temporal de 7 días y capacidad de recuperación habilitado, y configurado para tener un período de eliminación temporal de 14 días y la capacidad de recuperación desactivada:**

* *Opción 1 (recomendado):* establezca una directiva de retención de nivel de base de datos con un período de eliminación temporal de siete días y `MySpecialTable`capacidad de recuperación habilitada, y establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de 14 días y capacidad de recuperación deshabilitada para .

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

* *Opción 2:* Para cada tabla, establezca una directiva de retención de nivel de tabla con el período de eliminación temporal y la capacidad de recuperación deseados.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

**3. Establecer `MyTable1` `MyTable2` tablas , para tener un período `MySpecialTable` de eliminación temporal de 7 días, y tener mantener sus datos indefinidamente:**

* *Opción 1:* establezca una directiva de retención de nivel de base de datos con un período de eliminación temporal de siete días `MySpecialTable`y establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de 100 años (la directiva de retención predeterminada) para .

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}" // this sets the default retention policy
```

* *Opción 2:* `MyTable1`Para `MyTable2`tablas , , establezca una directiva de retención de nivel de tabla con el período `MySpecialTable` de eliminación temporal deseado de siete días y compruebe que la directiva de nivel de base de datos y de nivel de tabla no está establecida.

```kusto
.delete database MyDatabase policy retention   // optional, only if the database previously had its policy set
.delete table MySpecialTable policy retention   // optional, only if the table previously had its policy set
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
```

* *Opción 3*: `MyTable1` `MyTable2`Para tablas , , establezca una directiva de retención de nivel de tabla con el período de eliminación temporal deseado de siete días. Para `MySpecialTable`table , establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de 100 años (la directiva de retención predeterminada).

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}"
```
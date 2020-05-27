---
title: 'La Directiva de retención de Kusto controla cómo se quitan los datos: Azure Explorador de datos'
description: En este artículo se describe la Directiva de retención en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 3d854262a2a446f983f60c49a5c0ca02f6aa2ffe
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011381"
---
# <a name="retention-policy"></a>Directiva de retención

La Directiva de retención controla el mecanismo por el que se quitan automáticamente los datos de las tablas.
Esta eliminación suele ser útil para los datos que fluyen en una tabla continuamente cuya relevancia se basa en la edad. Por ejemplo, la Directiva de retención se puede usar para una tabla que contiene eventos de diagnóstico que pueden dejar de interesar después de dos semanas.

La Directiva de retención se puede configurar para una tabla específica o para una base de datos completa (en cuyo caso se aplica a todas las tablas de la base de datos que no invalidan la Directiva).

La configuración de una directiva de retención es importante para los clústeres que ingestan datos continuamente, lo que limitará los costos.

Los datos que están fuera de la Directiva de retención son válidos para su eliminación. Kusto no garantiza que se produzca la eliminación (por lo que los datos pueden "permanencia" incluso si se ha desencadenado la Directiva de retención).

La Directiva de retención se establece normalmente para limitar la antigüedad de los datos desde la ingesta (consulte [SoftDeletePeriod](#the-policy-object), a continuación).

> [!NOTE]
> * La hora de eliminación es imprecisa. El sistema garantiza que los datos no se eliminarán antes de que se supere el límite, pero la eliminación no es inmediata después de ese punto.
> * Un período de eliminación temporal de 0 se puede establecer como parte de una directiva de retención de nivel de tabla (pero no como parte de una directiva de retención de nivel de base de datos).
>   * Una vez hecho esto, los datos ingeridos no se confirmarán en la tabla de origen, evitando la necesidad de conservar los datos. Por consiguiente, `Recoverability` solo se puede establecer en `Disabled` . 
>   * Esta configuración es especialmente útil cuando los datos se introducen en una tabla.
>   Una [Directiva de actualización](updatepolicy.md) transaccional se utiliza para transformarla y redirigir la salida a otra tabla.

## <a name="the-policy-object"></a>El objeto de Directiva

Una directiva de retención incluye las siguientes propiedades:

* **SoftDeletePeriod**:
    * Un intervalo de tiempo para el que se garantiza que los datos se mantengan disponibles para la consulta, medidos desde el momento en que se ingesta.
    * Se establece de forma predeterminada en `100 years`.
    * Al modificar el período de eliminación temporal de una tabla o base de datos, el nuevo valor se aplica a los datos existentes y nuevos.
* **Capacidad de recuperación**:
    * Recuperación de datos (habilitada/deshabilitada) una vez eliminados los datos
    * De manera predeterminada, su valor es `Enabled`.
    * Si se establece en `Enabled` , los datos se recuperarán durante 14 días después de su eliminación temporal.

## <a name="control-commands"></a>Comandos de control

* Use [. Mostrar retención de directiva](../management/retention-policy.md) para mostrar la Directiva de retención actual para una base de datos o una tabla.
* Use [. Alter Policy Retention](../management/retention-policy.md) para cambiar la Directiva de retención actual de una base de datos o una tabla.

## <a name="defaults"></a>Valores predeterminados

De forma predeterminada, cuando se crea una base de datos o una tabla, no tiene definida una directiva de retención.
En los casos comunes, se crea la base de datos y, a continuación, la Directiva de retención se establece inmediatamente por su creador según los requisitos conocidos.
Al ejecutar un [comando show](../management/retention-policy.md) para la Directiva de retención de una base de datos o una tabla que no tiene su directiva establecida, `Policy` aparece como `null` .

La Directiva de retención predeterminada (con los valores predeterminados mencionados anteriormente) se puede aplicar con el siguiente comando:

```kusto
.alter database DatabaseName policy retention "{}"
.alter table TableName policy retention "{}"
```

Estos resultados con el siguiente objeto de Directiva se aplican a la base de datos o a la tabla:

```kusto
{
  "SoftDeletePeriod": "36500.00:00:00", "Recoverability":"Enabled"
}
```

La eliminación de la Directiva de retención de una base de datos o una tabla se puede realizar con el siguiente comando:

```kusto
.delete database DatabaseName policy retention
.delete table TableName policy retention
```

## <a name="examples"></a>Ejemplos

Dado que el clúster tiene una base de datos denominada `MyDatabase` , con tablas `MyTable1` `MyTable2` y`MySpecialTable`

**1. establecer todas las tablas de la base de datos para tener un período de eliminación temporal de 7 días y deshabilitar la capacidad de recuperación**:

* *Opción 1 (recomendado)*: establecer una directiva de retención de nivel de base de datos con un período de eliminación temporal de siete días y una capacidad de recuperación deshabilitada, y comprobar que no hay ninguna directiva de nivel de tabla establecida.

```kusto
.delete table MyTable1 policy retention        // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention        // optional, only if the table previously had its policy set
.delete table MySpecialTable policy retention  // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
```

* *Opción 2*: para cada tabla, establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de siete días y la capacidad de recuperación deshabilitada.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 7d recoverability = disabled
```

**2. configurar tablas `MyTable1` , `MyTable2` para tener un período de eliminación temporal de 7 días y capacidad de recuperación habilitada, y establecer `MySpecialTable` para tener un período de eliminación temporal de 14 días y la capacidad de recuperación deshabilitada**:

* *Opción 1 (recomendado)*: establecer una directiva de retención de nivel de base de datos con un período de eliminación temporal de siete días y una capacidad de recuperación habilitada, y establecer una directiva de retención de nivel de tabla con un período de eliminación temporal de 14 días y una capacidad de recuperación deshabilitada para `MySpecialTable` .

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

* *Opción 2*: para cada tabla, establezca una directiva de retención de nivel de tabla con el período de eliminación temporal deseado y la capacidad de recuperación.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

**3. configurar tablas `MyTable1` , `MyTable2` para tener un período de eliminación temporal de 7 días y `MySpecialTable` mantener sus datos indefinidamente**:

* *Opción 1*: establecer una directiva de retención de nivel de base de datos con un período de eliminación temporal de siete días y establecer una directiva de retención de nivel de tabla con un período de eliminación temporal de 100 años (la Directiva de retención predeterminada) para `MySpecialTable` .

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}" // this sets the default retention policy
```

* *Opción 2*: para las tablas `MyTable1` , `MyTable2` , establezca una directiva de retención de nivel de tabla con el período de eliminación temporal deseado de siete días y compruebe que la Directiva de nivel de base de datos y de nivel de tabla para `MySpecialTable` no está establecida.

```kusto
.delete database MyDatabase policy retention   // optional, only if the database previously had its policy set
.delete table MySpecialTable policy retention   // optional, only if the table previously had its policy set
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
```

* *Opción 3*: para las tablas `MyTable1` , `MyTable2` , establezca una directiva de retención de nivel de tabla con el período de eliminación temporal deseado de siete días. En tabla `MySpecialTable` , establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de 100 años (la Directiva de retención predeterminada).

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}"
```

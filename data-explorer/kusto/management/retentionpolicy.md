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
ms.openlocfilehash: 7c9bc193e739011a5f91d9bd5d4d8746a7ce2591
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780565"
---
# <a name="retention-policy"></a>Directiva de retención

La Directiva de retención controla el mecanismo que quita automáticamente los datos de las tablas. Resulta útil para quitar datos que fluyen continuamente en una tabla y cuya relevancia se basa en la edad. Por ejemplo, la Directiva se puede usar para una tabla que contiene eventos de diagnóstico que pueden dejar de interesar después de dos semanas.

La Directiva de retención se puede configurar para una tabla específica o para una base de datos completa.
A continuación, la Directiva se aplica a todas las tablas de la base de datos que no la invalidan.

La configuración de una directiva de retención es importante para los clústeres que ingestan datos continuamente, lo que limitará los costos.

Los datos que están fuera de la Directiva de retención son válidos para su eliminación. Kusto no garantiza que se produzca la eliminación. Los datos pueden "permanencia" incluso si se desencadena la Directiva de retención.

La Directiva de retención se establece normalmente para limitar la antigüedad de los datos desde la ingesta.
Para obtener más información, vea [SoftDeletePeriod](#the-policy-object).

> [!NOTE]
> * La hora de eliminación es imprecisa. El sistema garantiza que los datos no se eliminarán antes de que se supere el límite, pero la eliminación no es inmediata después de ese punto.
> * Un período de eliminación temporal de 0 se puede establecer como parte de una directiva de retención de nivel de tabla, pero no como parte de una directiva de retención de nivel de base de datos.
>   * Una vez hecho esto, los datos ingeridos no se confirmarán en la tabla de origen, evitando la necesidad de conservar los datos. Como resultado, `Recoverability` solo se puede establecer en `Disabled` .
>   * Esta configuración es especialmente útil cuando los datos se introducen en una tabla.
> Una [Directiva de actualización](updatepolicy.md) transaccional se utiliza para transformarla y redirigir la salida a otra tabla.

## <a name="the-policy-object"></a>El objeto de Directiva

Una directiva de retención incluye las siguientes propiedades:

* **SoftDeletePeriod**:
    * Intervalo de tiempo para el que se garantiza que los datos se mantengan disponibles para la consulta. El período se mide a partir de la hora en que se ingestan los datos.
    * Tiene como valor predeterminado `100 years`.
    * Al modificar el período de eliminación temporal de una tabla o base de datos, el nuevo valor se aplica a los datos existentes y nuevos.
* **Capacidad de recuperación**:
    * Recuperación de datos (habilitada/deshabilitada) una vez eliminados los datos.
    * Tiene como valor predeterminado `Enabled`.
    * Si se establece en `Enabled` , los datos se recuperarán durante 14 días después de su eliminación temporal.

## <a name="control-commands"></a>Comandos de control

* Use [. Mostrar retención de directiva](../management/retention-policy.md) para mostrar la Directiva de retención actual para una base de datos o una tabla.
* Use [. Alter Policy Retention](../management/retention-policy.md) para cambiar la Directiva de retención actual de una base de datos o una tabla.

## <a name="defaults"></a>Valores predeterminados

De forma predeterminada, cuando se crea una base de datos o una tabla, no tiene definida una directiva de retención. Normalmente, la base de datos se crea y, a continuación, la Directiva de retención se establece inmediatamente por su creador según los requisitos conocidos.
Al ejecutar un [comando show](../management/retention-policy.md) para la Directiva de retención de una base de datos o una tabla que no tenía su directiva establecida, `Policy` aparece como `null` .

La Directiva de retención predeterminada, con los valores predeterminados mencionados anteriormente, se puede aplicar con el siguiente comando.

```kusto
.alter database DatabaseName policy retention "{}"
.alter table TableName policy retention "{}"
```

El comando genera el siguiente objeto de directiva que se aplica a la base de datos o a la tabla.

```kusto
{
  "SoftDeletePeriod": "36500.00:00:00", "Recoverability":"Enabled"
}
```

La eliminación de la Directiva de retención de una base de datos o una tabla se puede realizar con el siguiente comando.

```kusto
.delete database DatabaseName policy retention
.delete table TableName policy retention
```

## <a name="examples"></a>Ejemplos

Para un clúster que tiene una base de datos denominada `MyDatabase` , con tablas `MyTable1` , `MyTable2` y `MySpecialTable` .

### <a name="soft-delete-period-of-seven-days-and-recoverability-disabled"></a>Período de eliminación temporal de siete días y capacidad de recuperación deshabilitada

Establezca todas las tablas de la base de datos para tener un período de eliminación temporal de siete días y deshabilitar la capacidad de recuperación.

* *Opción 1 (recomendado)*: establecer una directiva de retención de nivel de base de datos y comprobar que no se han establecido directivas de nivel de tabla.

```kusto
.delete table MyTable1 policy retention        // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention        // optional, only if the table previously had its policy set
.delete table MySpecialTable policy retention  // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
```

* *Opción 2*: para cada tabla, establezca una directiva de retención de nivel de tabla, con un período de eliminación temporal de siete días y la capacidad de recuperación deshabilitada.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 7d recoverability = disabled
```

### <a name="soft-delete-period-of-seven-days-and-recoverability-enabled"></a>Período de eliminación temporal de siete días y capacidad de recuperación habilitada

* Establezca las tablas `MyTable1` y `MyTable2` para tener un período de eliminación temporal de siete días y la capacidad de recuperación habilitada.
* `MySpecialTable`Se establece para tener un período de eliminación temporal de 14 días y la capacidad de recuperación deshabilitada.

* *Opción 1 (recomendado)*: establecer una directiva de retención de nivel de base de datos y establecer una directiva de retención de nivel de tabla.

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

* *Opción 2*: para cada tabla, establezca una directiva de retención de nivel de tabla, con el período de eliminación temporal pertinente y la capacidad de recuperación.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MyTable2 policy retention softdelete = 7d recoverability = disabled
.alter-merge table MySpecialTable policy retention softdelete = 14d recoverability = enabled
```

### <a name="soft-delete-period-of-seven-days-and-myspecialtable-keeps-its-data-indefinitely"></a>Período de eliminación temporal de siete días y `MySpecialTable` mantiene sus datos indefinidamente.

Establecer tablas `MyTable1` y `MyTable2` tener un período de eliminación temporal de siete días y `MySpecialTable` mantener sus datos indefinidamente.

* *Opción 1*: establecer una directiva de retención de nivel de base de datos y establecer una directiva de retención de nivel de tabla, con un período de eliminación temporal de 100 años, la Directiva de retención predeterminada, para `MySpecialTable` .

```kusto
.delete table MyTable1 policy retention   // optional, only if the table previously had its policy set
.delete table MyTable2 policy retention   // optional, only if the table previously had its policy set
.alter-merge database MyDatabase policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}" // this sets the default retention policy
```

* *Opción 2*: para las tablas `MyTable1` y `MyTable2` , establezca una directiva de retención de nivel de tabla y compruebe que la Directiva de nivel de base de datos y de nivel de tabla para `MySpecialTable` no está establecida.

```kusto
.delete database MyDatabase policy retention   // optional, only if the database previously had its policy set
.delete table MySpecialTable policy retention   // optional, only if the table previously had its policy set
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
```

* *Opción 3*: para las tablas `MyTable1` y `MyTable2` , establezca una directiva de retención de nivel de tabla. En tabla `MySpecialTable` , establezca una directiva de retención de nivel de tabla con un período de eliminación temporal de 100 años, la Directiva de retención predeterminada.

```kusto
.alter-merge table MyTable1 policy retention softdelete = 7d
.alter-merge table MyTable2 policy retention softdelete = 7d
.alter table MySpecialTable policy retention "{}"
```

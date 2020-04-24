---
title: 'evaluar el operador del complemento: Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de complemento de evaluación en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 1aae36df29abf705ba821abdc2d1da96e4635a60
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515749"
---
# <a name="evaluate-plugin-operator"></a>Operador evaluate plugin

Invoca una extensión de consulta del lado del servicio (plugin).

El `evaluate` operador es un operador tabular que proporciona la capacidad de invocar extensiones de lenguaje de consulta conocidas como **complementos.** Los complementos se pueden habilitar o deshabilitar (a diferencia de otras construcciones de lenguaje que siempre están disponibles) y no están "vinculados" por la naturaleza relacional del lenguaje (por ejemplo, pueden no tener un esquema de salida predefinido, determinado estáticamente).

**Sintaxis** 

[*T* `|`T `evaluate` ] [ *evaluateParameters* ] *PluginName* `(` [*PluginArg1* [`,` *PluginArg2*]...`)`

Donde:

* *T* es una entrada tabular opcional para el plugin. (Algunos plugins no toman ninguna entrada y actúan como una fuente de datos tabulares.)
* *PluginName* es el nombre obligatorio del complemento que se está invocando.
* *PluginArg1*, ... son los argumentos opcionales para el plugin.
* *evaluateParameters*: Cero o más parámetros (separados por espacios) en forma de *valor* de *nombre* `=` que controlan el comportamiento de la operación de evaluación y el plan de ejecución. Se admiten los siguientes parámetros: 

  |NOMBRE                |Valores                           |Descripción                                |
  |--------------------|---------------------------------|-------------------------------------------|
  |`hint.distribution` |`single`, `per_node`, `per_shard`| [Sugerencias de distribución](#distribution-hints) |

**Notas**

* Sintácticamente, `evaluate` se comporta de forma similar al [operador invoke](./invokeoperator.md), que invoca funciones tabulares.
* Los complementos proporcionados a través del operador evaluate no están vinculados por las reglas regulares de ejecución de consultas o evaluación de argumentos.
Los plugins específicos pueden tener restricciones específicas. Por ejemplo, los complementos cuyo esquema de salida depende de los datos (por ejemplo, [bag_unpack complemento)](./bag-unpackplugin.md)no se pueden utilizar al realizar consultas entre clústeres.

## <a name="distribution-hints"></a>Sugerencias de distribución

Las sugerencias de distribución especifican cómo se distribuirá la ejecución del complemento entre varios nodos del clúster. Cada plugin puede implementar un soporte diferente para la distribución. La documentación del plugin especifica las opciones de distribución soportadas por el plugin.

Valores posibles:

* `single`: Una sola instancia del plugin se ejecutará sobre todos los datos de la consulta.
* `per_node`: Si la consulta antes de la llamada del complemento se distribuye entre los nodos, entonces una instancia del plugin se ejecutará en cada nodo sobre los datos que contiene.
* `per_shard`: Si los datos antes de la llamada del plugin se distribuyen entre fragmentos, entonces una instancia del plugin se ejecutará sobre cada fragmento de los datos.
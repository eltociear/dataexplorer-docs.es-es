---
title: 'Instrucción de declaración de parámetros de consulta: Explorador de azure Data Explorer ( Explorador de datos de Azure) Microsoft Docs'
description: En este artículo se describe la instrucción de declaración de parámetros de consulta en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: b7193ada6967882306d9a977b6c90af8b247036d
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765501"
---
# <a name="query-parameters-declaration-statement"></a>Declaración de parámetros de consulta

::: zone pivot="azuredataexplorer"

Las consultas enviadas a Kusto pueden incluir, además del propio texto de consulta, un conjunto de pares nombre/valor **denominados parámetros**de consulta. A continuación, el texto de la consulta puede hacer referencia a uno o varios valores de parámetros de consulta especificando sus nombres y escribiendo a través de una instrucción de declaración de parámetros de **consulta.**

Los parámetros de consulta tienen dos usos principales:

1. Como mecanismo de protección contra ataques de **inyección.**
2. Como una forma de parametrizar las consultas.

En particular, las aplicaciones cliente que combinan la entrada proporcionada por el usuario en las consultas que luego envían a Kusto deben usar este mecanismo para proteger contra el equivalente de Kusto de ataques de [inyección SQL.](https://en.wikipedia.org/wiki/SQL_injection)

## <a name="declaring-query-parameters"></a>Declaración de parámetros de consulta

Para poder hacer referencia a parámetros de consulta, el texto de consulta (o las funciones que utiliza) debe declarar primero qué parámetro de consulta utiliza. Para cada parámetro, la declaración proporciona el nombre y el tipo escalar. Opcionalmente, el parámetro también puede tener un valor predeterminado que se usará si la solicitud no proporciona un valor concreto para el parámetro. Kusto, a continuación, analiza el valor del parámetro de consulta según sus reglas de análisis normales para ese tipo.

**Sintaxis**

`declare``query_parameters` `:` `=` `,` *Type1* *Name1* *DefaultValue1*Name1 Type1 [ DefaultValue1 ] [ ...] `(``);`

* *Name1*: el nombre de un parámetro de consulta utilizado en la consulta.
* *Tipo1*: El tipo correspondiente `string`(por `datetime`ejemplo, , , etc.) Los valores proporcionados por el usuario se codifican como cadenas, a Kusto aplicará el método de análisis adecuado al parámetro de consulta para obtener un valor fuertemente tipado.
* *DefaultValue1*: Un valor predeterminado opcional para el parámetro. Debe ser un literal del tipo escalar adecuado.

> [!NOTE]
> Al igual que las [funciones definidas por el usuario,](functions/user-defined-functions.md)los parámetros de consulta de tipo `dynamic` no pueden tener valores predeterminados.

**Ejemplos**

```kusto
declare query_parameters(UserName:string, Password:string);
print n=UserName, p=hash(Password)
```

```kusto
declare query_parameters(percentage:long = 90);
T | where Likelihood > percentage
```

## <a name="specifying-query-parameters-in-a-client-application"></a>Especificar parámetros de consulta en una aplicación cliente

Los nombres y valores de `string` los parámetros de consulta se proporcionan como valores por la aplicación que realiza la consulta. No se puede repetir ningún nombre.

La interpretación de los valores se realiza de acuerdo con la instrucción de declaración de parámetros de consulta. Cada valor se analiza como si fuera un literal en el cuerpo de una consulta según el tipo especificado por la instrucción de declaración de parámetros de consulta.

### <a name="rest-api"></a>API DE REST

Las aplicaciones cliente proporcionan parámetros de consulta a través de la `properties` ranura `Parameters`del objeto JSON del cuerpo de la solicitud, en un contenedor de propiedades anidado denominado . Por ejemplo, aquí está el cuerpo de una llamada a la API REST a Kusto que calcula la edad de algún usuario (presumiblemente haciendo que la aplicación pida al usuario su cumpleaños):

``` json
{
    "ns": null,
    "db": "myDB",
    "csl": "declare query_parameters(birthday:datetime); print strcat(\"Your age is: \", tostring(now() - birthday))",
    "properties": "{\"Options\":{},\"Parameters\":{\"birthday\":\"datetime(1970-05-11)\",\"courses\":\"dynamic(['Java', 'C++'])\"}}"
}
```

### <a name="kusto-net-sdk"></a>Kusto .NET SDK

Para proporcionar los nombres y valores de los parámetros de consulta al utilizar `ClientRequestProperties` la biblioteca de `HasParameter` `SetParameter`cliente `ClearParameter` Kusto .NET, se crea una nueva instancia del objeto y, a continuación, se utilizan los métodos , , y para manipular los parámetros de consulta. Tenga en cuenta que esta clase proporciona una `SetParameter`serie de sobrecargas fuertemente tipadas para ; internamente, generan el literal adecuado del lenguaje `string` de consulta y lo envían como a través de la API de REST, como se describió anteriormente. El propio texto de consulta debe [declarar los parámetros](#declaring-query-parameters)de consulta.

### <a name="kustoexplorer"></a>Kusto.Explorer

Para establecer los parámetros de consulta enviados al realizar una solicitud`ALT` + `P`al servicio, utilice el icono "llave" de **parámetros** de consulta ( ).

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

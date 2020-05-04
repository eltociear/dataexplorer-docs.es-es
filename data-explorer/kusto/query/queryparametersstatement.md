---
title: 'Instrucción de declaración de parámetros de consulta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la instrucción de declaración de parámetros de consulta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: a76755d04179b3d311e79798162c61db764455d7
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737817"
---
# <a name="query-parameters-declaration-statement"></a>Query Parameters (instrucción de declaración)

::: zone pivot="azuredataexplorer"

Las consultas enviadas a Kusto pueden incluir, además del texto de la consulta, un conjunto de pares nombre-valor denominados **parámetros de consulta**. Después, el texto de la consulta puede hacer referencia a uno o varios valores de parámetros de consulta mediante la especificación de sus nombres y su tipo a través de una **instrucción de declaración de parámetros de consulta**.

Los parámetros de consulta tienen dos usos principales:

1. Como mecanismo de protección contra **ataques por inyección**.
2. Como forma de parametrizar las consultas.

En concreto, las aplicaciones cliente que combinan la entrada proporcionada por el usuario en las consultas que envían a Kusto deben usar este mecanismo para protegerse frente al equivalente Kusto de ataques por [inyección de SQL](https://en.wikipedia.org/wiki/SQL_injection) .

## <a name="declaring-query-parameters"></a>Declarar parámetros de consulta

Para poder hacer referencia a los parámetros de consulta, el texto de la consulta (o las funciones que usa) debe declarar primero qué parámetro de consulta usa. Para cada parámetro, la Declaración proporciona el nombre y el tipo escalar. Opcionalmente, el parámetro también puede tener un valor predeterminado que se usará si la solicitud no proporciona un valor concreto para el parámetro. A continuación, Kusto analiza el valor del parámetro de consulta de acuerdo con sus reglas de análisis normales para ese tipo.

**Sintaxis**

`declare``query_parameters` `:` *Type1* `,` *Name1* `=` Nombre1 tipo1 [DefaultValue1] [...] *DefaultValue1* `(``);`

* *Nombre1*: el nombre de un parámetro de consulta que se usa en la consulta.
* *Tipo1*: el tipo correspondiente (por ejemplo `string`, `datetime`,, etc.) Los valores proporcionados por el usuario se codifican como cadenas; a Kusto, se aplicará el método parse adecuado al parámetro de consulta para obtener un valor fuertemente tipado.
* *DefaultValue1*: un valor predeterminado opcional para el parámetro. Debe ser un literal del tipo escalar adecuado.

> [!NOTE]
> Al igual que [las funciones definidas por](functions/user-defined-functions.md)el usuario `dynamic` , los parámetros de consulta de tipo no pueden tener valores predeterminados.

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

Los nombres y valores de los parámetros de consulta se `string` proporcionan como valores por parte de la aplicación que realiza la consulta. No se puede repetir ningún nombre.

La interpretación de los valores se realiza según la instrucción de declaración de los parámetros de consulta. Cada valor se analiza como si fuera un literal en el cuerpo de una consulta según el tipo especificado por la instrucción de declaración de parámetros de consulta.

### <a name="rest-api"></a>API DE REST

Los parámetros de consulta son proporcionados por las `properties` aplicaciones cliente a través de la ranura del objeto JSON del cuerpo de la solicitud, `Parameters`en un contenedor de propiedades anidadas denominado. Por ejemplo, este es el cuerpo de una llamada de API de REST a Kusto que calcula la edad de algún usuario (presumiblemente haciendo que la aplicación solicite al usuario su cumpleaños):

``` json
{
    "ns": null,
    "db": "myDB",
    "csl": "declare query_parameters(birthday:datetime); print strcat(\"Your age is: \", tostring(now() - birthday))",
    "properties": "{\"Options\":{},\"Parameters\":{\"birthday\":\"datetime(1970-05-11)\",\"courses\":\"dynamic(['Java', 'C++'])\"}}"
}
```

### <a name="kusto-net-sdk"></a>SDK de .NET para Kusto

Para proporcionar los nombres y valores de los parámetros de consulta cuando se usa la biblioteca de cliente de Kusto .net, se crea `ClientRequestProperties` una nueva instancia del objeto `HasParameter`y `SetParameter`, a `ClearParameter` continuación, se usan los métodos, y para manipular los parámetros de consulta. Tenga en cuenta que esta clase proporciona una serie de sobrecargas fuertemente tipadas `SetParameter`para; internamente, generan el literal adecuado del lenguaje de consulta y lo envían como un `string` a través de la API de REST, tal y como se ha descrito anteriormente. El propio texto de la consulta debe [declarar los parámetros de la consulta](#declaring-query-parameters).

### <a name="kustoexplorer"></a>Kusto.Explorer

Para establecer los parámetros de consulta enviados al realizar una solicitud al servicio, use el icono de la "llave inglesa"`ALT` + `P`de **los parámetros de consulta** ().

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

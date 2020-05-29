---
title: complemento de sql_request-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe sql_request complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 06cb62256b3dc433d375a23991ebc26b4f703c56
ms.sourcegitcommit: 3c86e21bd8a04eca39250a773d0e89ae05065b2e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2020
ms.locfileid: "84153648"
---
# <a name="sql_request-plugin"></a>sql_request plugin

::: zone pivot="azuredataexplorer"

  `evaluate``sql_request` `(` *ConnectionString* `,` *SqlQuery* [ `,` *SqlParameters* [ `,` *Opciones*]]`)`

El `sql_request` complemento envía una consulta SQL a un extremo de red SQL Server y devuelve el primer conjunto de filas en los resultados.

**Argumentos**

* *ConnectionString*: un `string` literal que indica la cadena de conexión que apunta al extremo de red SQL Server. Consulte a continuación los métodos válidos de autenticación y cómo especificar el punto de conexión de red.

* *SqlQuery*: un `string` literal que indica la consulta que se va a ejecutar en el extremo SQL. Debe devolver uno o más conjuntos de filas, pero solo el primero está disponible para el resto de la consulta Kusto.

* *SqlParameters*: un valor constante de tipo `dynamic` que contiene pares clave-valor que se pasan como parámetros junto con la consulta. Opcional.
  
* *Options*: un valor constante de tipo `dynamic` que contiene valores más avanzados como pares clave-valor. Actualmente solo `token` se puede establecer para pasar un token de acceso de AAD proporcionado por el autor de llamada que se reenvía al punto de conexión de SQL para la autenticación. Opcional.

**Ejemplos**

En el ejemplo siguiente se envía una consulta SQL a una base de datos de Azure SQL Database que recupera todos los registros de `[dbo].[Table]` y, a continuación, procesa los resultados en el lado de Kusto. La autenticación reutiliza el token de AAD del usuario que realiza la llamada.

Nota: este ejemplo no debe considerarse como una recomendación para filtrar o proyectar los datos de esta manera. Normalmente es preferible que las consultas SQL se construyan para devolver el conjunto de datos más pequeño posible, ya que actualmente el optimizador de Kusto no intenta optimizar las consultas entre Kusto y SQL.

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.windows.net,1433;'
    'Authentication="Active Directory Integrated";'
    'Initial Catalog=Fabrikam;',
  'select * from [dbo].[Table]')
| where Id > 0
| project Name
```

El ejemplo siguiente es idéntico al anterior, salvo que la autenticación de SQL se realiza mediante el nombre de usuario y la contraseña. Tenga en cuenta que, para la confidencialidad, usamos cadenas ofuscadas aquí.

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.windows.net,1433;'
    'Initial Catalog=Fabrikam;'
    h'User ID=USERNAME;'
    h'Password=PASSWORD;',
  'select * from [dbo].[Table]')
| where Id > 0
| project Name
```

En el ejemplo siguiente se envía una consulta SQL a una base de datos de Azure SQL Database recuperando todos los registros de `[dbo].[Table]` , mientras se anexa otra `datetime` columna y, a continuación, procesa los resultados en el lado de Kusto.
Especifica un parámetro SQL ( `@param0` ) que se va a usar en la consulta SQL.

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.windows.net,1433;'
    'Authentication="Active Directory Integrated";'
    'Initial Catalog=Fabrikam;',
  'select *, @param0 as dt from [dbo].[Table]',
  dynamic({'param0': datetime(2020-01-01 16:47:26.7423305)}))
| where Id > 0
| project Name
```

**Autenticación**

El complemento sql_request admite tres métodos de autenticación para el punto de conexión SQL Server:

* **Autenticación integrada de AAD** ( `Authentication="Active Directory Integrated"` ): este es el método preferido, en el que el usuario o la aplicación se autentica a través de AAD a Kusto y, a continuación, se usa el mismo token para tener acceso al extremo de red SQL Server.

* **Autenticación de nombre de usuario/contraseña** ( `User ID=...; Password=...;` ): se proporciona compatibilidad con este método cuando no se puede realizar la autenticación integrada de AAD. Evite este método cuando sea posible, ya que la información secreta se envía a través de Kusto.

* **Token de acceso de AAD** ( `dynamic({'token': h"eyJ0..."})` ): con este método de autenticación, el autor de la llamada genera el token de acceso y lo reenvía Kusto al punto de conexión de SQL. La cadena de conexión no debe incluir información de autenticación como `Authentication` , `User ID` o `Password` . En su lugar, el token de acceso se pasa como `token` propiedad en el `Options` argumento del complemento de sql_request.
     
> [!WARNING]
> Las cadenas de conexión y las consultas que incluyen información confidencial o información que debe protegerse deben ofuscarse para que se omitan en cualquier seguimiento de Kusto.
> Vea [literales de cadena ofuscados](scalar-data-types/string.md#obfuscated-string-literals) para obtener más detalles.

**Cifrado y validación de servidor**

Las siguientes propiedades de conexión se fuerzan al conectarse a un punto de conexión de red SQL Server, por motivos de seguridad:

* `Encrypt`está establecido en `true` incondicionalmente.
* `TrustServerCertificate`está establecido en `false` incondicionalmente.

Como resultado, el SQL Server se debe configurar con un certificado de servidor SSL/TLS válido.

**Especificación del punto de conexión de red**

Es obligatorio especificar el extremo de red de SQL como parte de la cadena de conexión.
La sintaxis adecuada es:

`Server``=` `tcp:` *FQDN* [ `,` *Puerto*]

Dónde:

* *FQDN* es el nombre de dominio completo del punto de conexión.

* *Port* es el puerto TCP del extremo. De forma predeterminada, `1433` se supone.

> [!NOTE]
> No se admiten otras formas de especificar el punto de conexión de red.
> No se puede omitir, por ejemplo, el prefijo `tcp:` aunque sea posible hacerlo al usar las bibliotecas de cliente SQL mediante programación.



::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

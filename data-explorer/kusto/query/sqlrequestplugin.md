---
title: sql_request complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe sql_request complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: f0c0837c6bb8e4dcd3cf2e28af18d02c19edb676
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766177"
---
# <a name="sql_request-plugin"></a>sql_request plugin

::: zone pivot="azuredataexplorer"

  `evaluate``sql_request` `,` `,` *SqlParameters* `,` *Options* *SqlQuery* *ConnectionString* ConnectionString SqlQuery [ SqlParameters [ Opciones ]] `(``)`

El `sql_request` complemento envía una consulta SQL a un extremo de red de SQL Server y devuelve el primer conjunto de filas de los resultados.

**Argumentos**

* *ConnectionString*: `string` literal que indica la cadena de conexión que apunta al extremo de red de SQL ServerSQL Server . Consulte a continuación los métodos válidos de autenticación y cómo especificar el punto de conexión de red.

* *SqlQuery*: `string` literal que indica la consulta que se va a ejecutar en el extremo SQL. Debe devolver uno o varios conjuntos de filas, pero solo el primero está disponible para el resto de la consulta de Kusto.

* *SqlParameters*: Un valor `dynamic` constante de tipo que contiene pares clave-valor para pasar como parámetros junto con la consulta. Opcional.
  
* *Opciones*: Un valor `dynamic` constante de tipo que contiene ajustes más avanzados como pares clave-valor. Actualmente `token` solo se puede establecer, para pasar un token de acceso de AAD proporcionado por el autor de la llamada que se reenvía al punto de conexión SQL para la autenticación. Opcional.

**Ejemplos**

En el ejemplo siguiente se envía una consulta SQL `[dbo].[Table]`a una base de datos de Azure SQL DB que recupera todos los registros y, a continuación, se procesan los resultados en el lado de Kusto. La autenticación reutiliza el token de AAD del usuario que realiza la llamada.

Nota: Este ejemplo no se debe tomar como una recomendación para filtrar/proyectar datos de esta manera. Normalmente es preferible que las consultas SQL se construyan para devolver el conjunto de datos más pequeño posible, ya que actualmente el optimizador de Kusto no intenta optimizar las consultas entre Kusto y SQL.

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.windows.net,1433;'
    'Authentication="Active Directory Integrated";'
    'Initial Catalog=Fabrikam;',
  'select * from [dbo].[Table]')
| where Id > 0
| project Name
```

El ejemplo siguiente es idéntico al anterior, excepto que la autenticación SQL se realiza mediante nombre de usuario/contraseña. Tenga en cuenta que para la confidencialidad, utilizamos cadenas ofuscadas aquí.

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

**Autenticación**

El complemento sql_request admite tres métodos de autenticación en el punto de conexión de SQL Server:

* **Autenticación integrada** de`Authentication="Active Directory Integrated"`AAD ( ): este es el método preferido, en el que el usuario o la aplicación se autentica a través de AAD a Kusto y, a continuación, se usa el mismo token para tener acceso al extremo de red de SQL ServerSQL Server .

* Autenticación de nombre`User ID=...; Password=...;`de **usuario/contraseña** ( ): Se proporciona compatibilidad con este método cuando no se puede realizar la autenticación integrada de AAD. Evite este método, cuando sea posible, ya que la información secreta se envía a través de Kusto.

* **Token** de acceso`dynamic({'token': h"eyJ0..."})`de AAD ( ): Con este método de autenticación, el autor de la llamada genera el token de acceso y Kusto lo reenvía al punto de conexión SQL. La cadena de conexión no `Authentication` `User ID`debe `Password`incluir información de autenticación como , , o . En su lugar, el `token` token `Options` de acceso se pasa como propiedad en el argumento del complemento sql_request.
     
> [!WARNING]
> Las cadenas de conexión y las consultas que incluyan información confidencial o información que se debe proteger deben ocultarse para que se omitan de cualquier seguimiento de Kusto.
> Consulte literales de [cadena ofuscados](scalar-data-types/string.md#obfuscated-string-literals) para obtener más detalles.

**Cifrado y validación del servidor**

Las siguientes propiedades de conexión se fuerzan al conectarse a un extremo de red de SQL Server, por motivos de seguridad:

* `Encrypt`se establece `true` incondicionalmente.
* `TrustServerCertificate`se establece `false` incondicionalmente.

Como resultado, SQL ServerSQL Server debe configurarse con un certificado de servidor SSL/TLS válido.

**Especificar el punto final de red**

Especificar el extremo de red SQL como parte de la cadena de conexión es obligatorio.
La sintaxis adecuada es:

`Server``=` *FQDN* `,` *Port*FQDN [ Puerto ] `tcp:`

Donde:

* *FQDN* es el nombre de dominio completo del punto de conexión.

* *El puerto* es el puerto TCP del punto final. De forma `1433` predeterminada, se supone.

> [!NOTE]
> No se admiten otras formas de especificar el punto de conexión de red.
> No se puede omitir, `tcp:` por ejemplo, el prefijo aunque sea posible hacerlo cuando se usan las bibliotecas de cliente SQL mediante programación.



::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

---
title: 'Consultas entre bases de datos y entre clústeres: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las consultas entre bases de datos y entre clústeres en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 8bb0cf2674bae801fb98d14d93518f5617af6807
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766065"
---
# <a name="cross-database-and-cross-cluster-queries"></a>Consultas entre bases de datos y entre clústeres

::: zone pivot="azuredataexplorer"

Cada consulta de Kusto funciona en el contexto del clúster actual y la base de datos predeterminada.
* En [Kusto Explorer,](../tools/kusto-explorer.md) la base de datos predeterminada es la seleccionada en el [panel Conexiones](../tools/kusto-explorer.md#connections-panel) y el clúster actual es la conexión que contiene esa base de datos
* Cuando se utiliza la biblioteca de cliente de [Kusto,](../api/netfx/about-kusto-data.md) el clúster actual y la base de datos predeterminada se especifican mediante las `Data Source` propiedades y `Initial Catalog` de las cadenas de conexión de [Kusto,](../api/connection-strings/kusto.md) respectivamente.

## <a name="queries"></a>Consultas
Para tener acceso a tablas desde cualquier base de datos que no sea la predeterminada, se debe utilizar la sintaxis de *nombre completo:* Para tener acceso a la base de datos en el clúster actual:
```kusto
database("<database name>").<table name>
```
Base de datos en clúster remoto:
```kusto
cluster("<cluster name>").database("<database name>").<table name>
```

*nombre* de la base de datos distingue mayúsculas de minúsculas

*el nombre* del clúster no distingue mayúsculas de minúsculas y puede ser de una de las siguientes formas:
* URL bien formada: `http://contoso.kusto.windows.net:1234/`por ejemplo, solo se admiten esquemas HTTP y HTTPS.
* Nombre de dominio completo (FQDN): por ejemplo `contoso.kusto.windows.net` , que será equivalente a`https://`**`contoso.kusto.windows.net`**`:443/`
* Nombre corto (nombre de host [y región] `contoso` sin la `https://` **`contoso`** `.kusto.windows.net:443/`parte `contoso.westus` del dominio): por ejemplo - que se interpreta como , o - que se interpreta como`https://`**`contoso.westus`**`.kusto.windows.net:443/`

> [!NOTE]
> El acceso entre bases de datos está sujeto a las comprobaciones de permisos habituales.
> Para excute una consulta, debe tener permiso de lectura en la base de datos predeterminada y en todas las demás bases de datos a las que se hace referencia en la consulta (en los clústeres actuales y remotos).

*El nombre completo* se puede utilizar en cualquier contexto en el que se pueda utilizar un nombre de tabla.
Todas las siguientes son válidas:

```kusto
database("OtherDb").Table | where ...

union Table1, cluster("OtherCluster").database("OtherDb").Table2 | project ...

database("OtherDb1").Table1 | join cluster("OtherCluster").database("OtherDb2").Table2 on Key | join Table3 on Key | extend ...
```

Cuando *el nombre completo* aparece como un operando del operador de [unión,](./unionoperator.md)se pueden usar comodines para especificar varias tablas y varias bases de datos. Los comodines no están permitidos en los nombres de clúster:

```kusto
union withsource=TableName *, database("OtherDb*").*Table, cluster("OtherCluster").database("*").*
```

> [!NOTE]
>* El nombre de la base de datos predeterminada también es una coincidencia potencial, por lo que database("&#42;").* especifica todas las tablas de todas las bases de datos, incluido el valor predeterminado.
>* ¿Cómo afectan los cambios de esquema a las consultas entre clústeres, consulte [Consultas entre clústeres y cambios](../concepts/crossclusterandschemachanges.md) de esquema

## <a name="access-restriction"></a>Restricción de acceso 
Los nombres o patrones calificados también se pueden incluir en la instrucción de [acceso restrict](./restrictstatement.md) (no se permiten comodines en los nombres de clúster)
```kusto
restrict access to (my*, database("MyOther*").*, cluster("OtherCluster").database("my2*").*);
```

Lo anterior restringirá el acceso de la consulta a los siguientes elementos:

* Cualquier nombre de entidad que empiece por *mi...* en la base de datos predeterminada. 
* Cualquier tabla de todas las bases de datos denominada *MyOther...* del clúster actual.
* Cualquier tabla de todas las bases de datos denominadas *my2...* en el *clúster OtherCluster.kusto.windows.net*.

## <a name="functions-and-views"></a>Funciones y vistas

Las funciones y vistas (persistentes y creadas en línea) pueden hacer referencia a tablas a través de los límites de la base de datos y del clúster. Lo siguiente es válido:

```kusto
let MyView = Table1 join database("OtherDb").Table2 on Key | join cluster("OtherCluster").database("SomeDb").Table3 on Key;
MyView | where ...
```

Se puede acceder a las funciones y vistas persistentes desde otra base de datos del mismo clúster:

Función tabular (vista) en: `OtherDb`

```kusto
.create function MyView(v:string) { Table1 | where Column1 has v ...  }  
```

Función escalar `OtherDb`en:
```kusto
.create function MyCalc(a:double, b:double, c:double) { (a + b) / c }  
```

En la base de datos predeterminada:

```kusto
database("OtherDb").MyView("exception") | extend CalCol=database("OtherDb").MyCalc(Col1, Col2, Col3) | limit 10
```

## <a name="limitations-of-cross-cluster-function-calls"></a>Limitaciones de las llamadas a funciones entre clústeres

Se puede hacer referencia a funciones o vistas tabulares entre clústeres. Se aplica la siguiente limitación:

1. La función remota debe devolver un esquema tabular. Solo se puede acceder a las funciones escalares en el mismo clúster.
2. La función remota solo puede aceptar parámetros escalares. Solo se puede tener acceso a las funciones que obtienen uno o varios argumentos de tabla en el mismo clúster.
3. El esquema de la función remota debe ser conocido e invariable de sus parámetros (consulte también [consultas entre clústeres y cambios](../concepts/crossclusterandschemachanges.md)de esquema).

La siguiente llamada entre clústeres es **válida:**

```kusto
cluster("OtherCluster").database("SomeDb").MyView("exception") | count
```

La siguiente consulta llama a `MyCalc`la función escalar remota .
Esto está violando la regla #1, por lo tanto no es **válido:**

```kusto
MyTable | extend CalCol=cluster("OtherCluster").database("OtherDb").MyCalc(Col1, Col2, Col3) | limit 10
```

La siguiente consulta `MyCalc` llama a la función remota y proporciona un parámetro tabular.
Esto es violar la regla #2, por lo tanto no es **válido:**

```kusto
cluster("OtherCluster").database("OtherDb").MyCalc(datatable(x:string, y:string)["x","y"] ) 
```

La siguiente consulta `SomeTable` llama a la función `tablename`remota que tiene una salida de esquema variable basada en el parámetro .
Esto está violando la regla #3, por lo tanto no es **válido:**

Función `OtherDb`tabular en :
```kusto
.create function SomeTable(tablename:string) { table(tablename)  }  
```

En la base de datos predeterminada:
```kusto
cluster("OtherCluster").database("OtherDb").SomeTable("MyTable")
```

La siguiente consulta `GetDataPivot` llama a la función remota que tiene una salida de esquema variable basada en los datos ([pivot() plugin](pivotplugin.md) tiene salida dinámica).
Esto está violando la regla #3, por lo tanto no es **válido:**

Función `OtherDb`tabular en :
```kusto
.create function GetDataPivot() { T | evaluate pivot(PivotColumn) }  
```

Función tabular en la base de datos predeterminada:
```kusto
cluster("OtherCluster").database("OtherDb").GetDataPivot()
```

## <a name="displaying-data"></a>Mostrar datos

Las instrucciones que devuelven datos al cliente están implícitamente limitadas por el número `take` de registros devueltos, incluso si no hay ningún uso específico del operador. Para eliminar este límite, use la opción de solicitud de cliente `notruncation` .

Para mostrar los datos en forma gráfica, utilice el [operador de renderización](renderoperator.md).

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end

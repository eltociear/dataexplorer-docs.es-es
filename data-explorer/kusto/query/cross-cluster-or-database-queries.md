---
title: 'Consultas entre clústeres & entre bases de datos: Azure Explorador de datos'
description: En este artículo se describen las consultas entre bases de datos y entre clústeres en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: bb25fd556ab59dc5bdf5c533435f99deb6b32fdb
ms.sourcegitcommit: da7c699bb62e1c4564f867d4131d26286c5223a8
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2020
ms.locfileid: "83404234"
---
# <a name="cross-database-and-cross-cluster-queries"></a>Consultas entre bases de datos y entre clústeres

::: zone pivot="azuredataexplorer"

Cada consulta de Kusto funciona en el contexto del clúster actual y la base de datos predeterminada.
* En el [Explorador de Kusto](../tools/kusto-explorer.md), la base de datos predeterminada es la seleccionada en el [Panel conexiones](../tools/kusto-explorer.md#connections-panel) y el clúster actual es la conexión que contiene la base de datos.
* Al usar la [biblioteca de cliente de Kusto](../api/netfx/about-kusto-data.md), el clúster actual y la base de datos predeterminada se especifican mediante las `Data Source` propiedades y `Initial Catalog` de las [cadenas de conexión Kusto](../api/connection-strings/kusto.md) , respectivamente.

## <a name="queries"></a>Consultas
Para tener acceso a las tablas de cualquier base de datos que no sea la predeterminada, se debe usar la sintaxis de *nombre completo* : para tener acceso a la base de datos del clúster actual:
```kusto
database("<database name>").<table name>
```
Base de datos en clúster remoto:
```kusto
cluster("<cluster name>").database("<database name>").<table name>
```

*el nombre* de la base de datos distingue mayúsculas de minúsculas

el *nombre del clúster* no distingue entre mayúsculas y minúsculas y puede tener una de las siguientes formas:
* Dirección URL con formato correcto, como `http://contoso.kusto.windows.net:1234/` . Solo se admiten los esquemas HTTP y HTTPS.
* Nombre de dominio completo (FQDN), como `contoso.kusto.windows.net` . Esta cadena es equivalente a `https://` **`contoso.kusto.windows.net`** `:443/` .
* Nombre corto (nombre de host [y región] sin la parte de dominio), como `contoso` o `contoso.westus` . Estas cadenas se interpretan como `https://` **`contoso`** `.kusto.windows.net:443/` y `https://` **`contoso.westus`** `.kusto.windows.net:443/` .

> [!NOTE]
> El acceso entre bases de datos está sujeto a las comprobaciones de permisos habituales.
> Para realizar una consulta, debe tener permiso de lectura en la base de datos predeterminada y en todas las demás bases de datos a las que se haga referencia en la consulta (en los clústeres actuales y remotos).

*El nombre completo* se puede usar en cualquier contexto en el que se pueda usar un nombre de tabla.
Los siguientes son válidos:

```kusto
database("OtherDb").Table | where ...

union Table1, cluster("OtherCluster").database("OtherDb").Table2 | project ...

database("OtherDb1").Table1 | join cluster("OtherCluster").database("OtherDb2").Table2 on Key | join Table3 on Key | extend ...
```

Cuando el *nombre completo* aparece como un operando del [operador Union](./unionoperator.md), se pueden usar comodines para especificar varias tablas y varias bases de datos. No se permiten caracteres comodín en los nombres de clúster:

```kusto
union withsource=TableName *, database("OtherDb*").*Table, cluster("OtherCluster").database("*").*
```

> [!NOTE]
>* El nombre de la base de datos predeterminada también es una posible coincidencia, por lo que la base de datos ("&#42;"). * especifica todas las tablas de todas las bases de datos, incluida la predeterminada.
>* Cómo afectan los cambios de esquema a las consultas entre clústeres, vea [consultas entre clústeres y cambios de esquema](../concepts/crossclusterandschemachanges.md)

## <a name="access-restriction"></a>Restricción de acceso 
Los nombres o patrones completos también se pueden incluir en la instrucción de [restricción de acceso](./restrictstatement.md) (no se permiten caracteres comodín en los nombres de clúster)
```kusto
restrict access to (my*, database("MyOther*").*, cluster("OtherCluster").database("my2*").*);
```

Lo anterior restringirá el acceso de la consulta a las siguientes entidades:

* Cualquier nombre de entidad que empiece por *My...* en la base de datos predeterminada. 
* Cualquier tabla de todas las bases de datos denominadas mi *..* . del clúster actual.
* Cualquier tabla de todas las bases de datos denominada *MY2...* en el clúster *OtherCluster.kusto.Windows.net*.

## <a name="functions-and-views"></a>Funciones y vistas

Las funciones y vistas (persistentes y creadas en línea) pueden hacer referencia a tablas en los límites de la base de datos y del clúster. El código siguiente es válido:

```kusto
let MyView = Table1 join database("OtherDb").Table2 on Key | join cluster("OtherCluster").database("SomeDb").Table3 on Key;
MyView | where ...
```

Se puede tener acceso a las funciones y vistas persistentes desde otra base de datos del mismo clúster:

Función tabular (vista) en `OtherDb` :

```kusto
.create function MyView(v:string) { Table1 | where Column1 has v ...  }  
```

Función escalar en `OtherDb` :
```kusto
.create function MyCalc(a:double, b:double, c:double) { (a + b) / c }  
```

En la base de datos predeterminada:

```kusto
database("OtherDb").MyView("exception") | extend CalCol=database("OtherDb").MyCalc(Col1, Col2, Col3) | limit 10
```

## <a name="limitations-of-cross-cluster-function-calls"></a>Limitaciones de las llamadas a funciones entre clústeres

En los clústeres se puede hacer referencia a las funciones tabulares o vistas. Se aplican las siguientes limitaciones:

1. La función remota debe devolver el esquema tabular. Solo se puede tener acceso a las funciones escalares en el mismo clúster.
2. La función remota solo puede aceptar parámetros escalares. Solo se puede tener acceso a las funciones que obtienen uno o más argumentos de tabla en el mismo clúster.
3. El esquema de la función remota debe ser conocido e invariable de sus parámetros (vea también [consultas entre clústeres y cambios de esquema](../concepts/crossclusterandschemachanges.md)).

La siguiente llamada entre clústeres es **válida**:

```kusto
cluster("OtherCluster").database("SomeDb").MyView("exception") | count
```

La consulta siguiente llama a la función escalar remota `MyCalc` .
Esta llamada infringe la regla #1, por lo que **no es válida**:

```kusto
MyTable | extend CalCol=cluster("OtherCluster").database("OtherDb").MyCalc(Col1, Col2, Col3) | limit 10
```

La consulta siguiente llama a la función remota `MyCalc` y proporciona un parámetro tabular.
Esta llamada infringe la regla #2, por lo que **no es válida**:

```kusto
cluster("OtherCluster").database("OtherDb").MyCalc(datatable(x:string, y:string)["x","y"] ) 
```

La consulta siguiente llama a la función remota `SomeTable` que tiene una salida de esquema variable basada en el parámetro `tablename` .
Esta llamada infringe la regla #3, por lo que **no es válida**:

Función tabular en `OtherDb` :
```kusto
.create function SomeTable(tablename:string) { table(tablename)  }  
```

En la base de datos predeterminada:
```kusto
cluster("OtherCluster").database("OtherDb").SomeTable("MyTable")
```

La consulta siguiente llama a la función remota `GetDataPivot` que tiene una salida de esquema variable basada en el complemento de datos ([Pivot ()](pivotplugin.md) tiene una salida dinámica).
Esta llamada infringe la regla #3, por lo que **no es válida**:

Función tabular en `OtherDb` :
```kusto
.create function GetDataPivot() { T | evaluate pivot(PivotColumn) }  
```

Función tabular en la base de datos predeterminada:
```kusto
cluster("OtherCluster").database("OtherDb").GetDataPivot()
```

## <a name="displaying-data"></a>Mostrar datos

Las instrucciones que devuelven datos al cliente están limitadas implícitamente por el número de registros devueltos, incluso si no hay ningún uso específico del `take` operador. Para eliminar este límite, use la opción de solicitud de cliente `notruncation` .

Para mostrar los datos en formato gráfico, use el [operador Render](renderoperator.md).

::: zone-end

::: zone pivot="azuremonitor"

Las consultas entre bases de datos y entre clústeres no se admiten en Azure Monitor.

::: zone-end

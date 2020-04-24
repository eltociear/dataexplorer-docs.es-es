---
title: 'Biblioteca de cliente de Kusto: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la biblioteca de cliente de Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/15/2020
ms.openlocfilehash: 8a3ab9bb3727efdc80e1887ab802f2ad4e3c7cce
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81502608"
---
# <a name="kusto-client-library"></a>Biblioteca de clientes de Kusto
    
El SDK de cliente de Kusto (Kusto.Data) expone una API mediante programación similar a ADO.NET, por lo que su uso debe sentirse natural para aquellos experimentados con .NET. Puede crear un cliente`ICslQueryProvider`de consulta (`ICslAdminProvider`) o un proveedor de comandos de control ( ) a partir de un objeto de cadena de conexión que apunte al servicio de motor de Kusto, la base de datos, el método de autenticación, etc. A continuación, puede emitir consultas de datos o comandos de control especificando la cadena de `IDataReader` lenguaje de consulta Kusto adecuada y recuperar una o varias tablas de datos a través del objeto devuelto.

De forma más concreta, para crear un cliente similar a ADO.NET que `Kusto.Data.Net.Client.KustoClientFactory` permita consultas en Kusto, utilice métodos estáticos en la clase. Estos toman la cadena de conexión y crean un objeto de cliente seguro para subprocesos, desechable. (Se recomienda encarecidamente que el código de cliente no cree "demasiadas" instancias de este objeto. En su lugar, el código de cliente debe crear un objeto por cadena de conexión y mantenerlo durante el tiempo que sea necesario.) Esto permite que el objeto de cliente almacene en caché eficazmente los recursos.

En general, todos los métodos de los clientes son seguros para subprocesos con dos excepciones: `Dispose`, y propiedades de establecedor. Para obtener resultados coherentes, no invoque ninguno de los métodos simultáneamente.

Estos son algunos ejemplos. Puede encontrar muestras adicionales [aquí](https://github.com/Azure/azure-kusto-samples-dotnet/tree/master/client).

**Ejemplo: Contando filas**
 
El código siguiente muestra el recuento `StormEvents` de las `Samples`filas de una tabla denominada en una base de datos denominada:

```csharp
var client = Kusto.Data.Net.Client.KustoClientFactory.CreateCslQueryProvider("https://help.kusto.windows.net/Samples;Fed=true");
var reader = client.ExecuteQuery("StormEvents | count");
// Read the first row from reader -- it's 0'th column is the count of records in MyTable
// Don't forget to dispose of reader when done.
```

**Ejemplo: Obtener información de diagnóstico del clúster de Kusto**

```csharp
var kcsb = new KustoConnectionStringBuilder(cluster URI here). WithAadUserPromptAuthentication();
using (var client = KustoClientFactory.CreateCslAdminProvider(kcsb))
{
    var diagnosticsCommand = CslCommandGenerator.GenerateShowDiagnosticsCommand();
    var objectReader = new ObjectReader<DiagnosticsShowCommandResult>(client.ExecuteControlCommand(diagnosticsCommand));
    DiagnosticsShowCommandResult diagResult = objectReader.ToList().FirstOrDefault();
    // DO something with the diagResult    
}
```



## <a name="the-kustoclientfactory-client-factory"></a>La fábrica de clientes KustoClientFactory

La clase `Kusto.Data.Net.Client.KustoClientFactory` estática proporciona el punto de entrada principal para los autores de código de cliente que utiliza Kusto. Proporciona los siguientes métodos estáticos importantes:

|Método                                      |Devuelve                                |Se usa para                                                      |
|--------------------------------------------|---------------------------------------|--------------------------------------------------------------|
|`CreateCslQueryProvider`                    |`ICslQueryProvider`                    |Envío de consultas a un clúster de motores de Kusto.                    |
|`CreateCslAdminProvider`                    |`ICslAdminProvider`                    |Envío de comandos de control a un clúster de Kusto (de cualquier tipo).    |
|`CreateRedirectProvider`                    |`IRedirectProvider`                    |Creación de un mensaje de respuesta HTTP de redirección para una solicitud de Kusto.|


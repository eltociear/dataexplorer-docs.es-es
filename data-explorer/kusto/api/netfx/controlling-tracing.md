---
title: Control o supresión del seguimiento del lado del cliente del SDK de Kusto - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Controlar o suprimir el seguimiento del lado del cliente de Kusto SDK en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 65964d7e990cdb639bd5bfe319d11874ea3de15d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81502744"
---
# <a name="controlling-or-suppressing-kusto-sdk-client-side-tracing"></a>Control o supresión del seguimiento del lado del cliente de Kusto SDK

Las bibliotecas de cliente de Kusto utilizan una plataforma común para el seguimiento. La plataforma utiliza un gran`System.Diagnostics.TraceSource`número de orígenes de seguimiento (`System.Diagnostics.Trace.Listeners`), cada uno conectado al conjunto predeterminado de agentes de escucha de seguimiento ( ) durante su construcción.

Una implicación de esto es que si una `System.Diagnostics.Trace` aplicación tiene agentes `app.config` de escucha de seguimiento asociados con la instancia predeterminada (por ejemplo, a través de su archivo), las bibliotecas de cliente de Kusto emitirán seguimientos a esos agentes de escucha.

Este comportamiento se puede suprimir o controlar mediante programación o a través de un archivo de configuración.

## <a name="suppress-tracing-programmatically"></a>Suprimir el seguimiento mediante programación

Para suprimir el seguimiento de las bibliotecas de cliente de Kusto mediante programación, invoque el siguiente fragmento de código desde el principio al cargar la biblioteca relevante:

```csharp
Kusto.Cloud.Platform.Utils.TraceSourceManager.SetTraceVerbosityForAll(
    Kusto.Cloud.Platform.Utils.TraceVerbosity.Fatal
    );
```

## <a name="suppressing-tracing-by-using-a-config-file"></a>Supresión del seguimiento mediante un archivo de configuración

Para suprimir el seguimiento de las bibliotecas de cliente `Kusto.Cloud.Platform.dll.tweaks` de Kusto `Kusto.Data` a través de un archivo de configuración, modifique el archivo (que se incluye con la biblioteca) para que ahora se diga el "tweak" adecuado:

```xml
    <!-- Overrides the default trace verbosity level -->
    <add key="Kusto.Cloud.Platform.Utils.Tracing.OverrideTraceVerbosityLevel" value="0" />
```

(Tenga en cuenta que para que el ajuste surta `key`efecto no debe haber ningún signo menos en el valor de .)

Un medio alternativo es hacer lo siguiente:

```csharp
Kusto.Cloud.Platform.Utils.Anchor.Tweaks.SetProgrammaticAppSwitch(
    "Kusto.Cloud.Platform.Utils.Tracing.OverrideTraceVerbosityLevel",
    "0"
    );
```

## <a name="how-to-enable-the-kusto-client-libraries-tracing"></a>Cómo habilitar el seguimiento de las bibliotecas de cliente de Kusto

Para habilitar el seguimiento fuera de las bibliotecas de cliente de Kusto, habilite el seguimiento de .NET en el archivo app.config de la aplicación. Por ejemplo, suponiendo `MyApp.exe` que la aplicación está utilizando la biblioteca de cliente Kusto.Data. A continuación, `MyApp.exe.config` al cambiar el archivo para incluir lo siguiente, se habilitará el seguimiento de Kusto.Data la próxima vez que se inicie la aplicación:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.diagnostics>
    <trace indentsize="4">
      <listeners>
        <add type="Kusto.Cloud.Platform.Utils.RollingCsvTraceListener2, Kusto.Cloud.Platform" name="RollingCsvTraceListener" initializeData="RollingLogs" />
        <remove name="Default" />
      </listeners>
    </trace>
  </system.diagnostics>
</configuration>
``` 

Esto configurará un agente de escucha de seguimiento que `RollingLogs` escribe en archivos CSV en un subdirectorio denominado ubicado en el directorio del proceso. (Por supuesto, cualquier . También se puede utilizar la clase de escucha de seguimiento compatible con NET.) 

## <a name="how-to-enable-the-aad-client-libraries-adal-tracing"></a>Cómo habilitar el seguimiento de las bibliotecas de cliente de AAD (ADAL)

Una vez habilitado el seguimiento de las bibliotecas de cliente de Kusto, también lo son las bibliotecas de cliente de AAD emitidas por las bibliotecas de cliente de AAD (las bibliotecas de cliente de Kusto configuran automáticamente el seguimiento ADAL)


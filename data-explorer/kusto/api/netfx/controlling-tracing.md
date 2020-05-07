---
title: 'Control o supresión del seguimiento del lado cliente del SDK de Kusto: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el control o la supresión del seguimiento del lado cliente del SDK de Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 10/23/2018
ms.openlocfilehash: cbda69063e3b1a20549dbadb4641fc9fd3f51f57
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82862061"
---
# <a name="controlling-or-suppressing-kusto-sdk-client-side-tracing"></a>Control o supresión del seguimiento del lado cliente del SDK de Kusto

Las bibliotecas de cliente de Kusto usan una plataforma común para el seguimiento. La plataforma usa un gran número de orígenes de seguimiento`System.Diagnostics.TraceSource`(), cada uno conectado al conjunto predeterminado de agentes de escucha`System.Diagnostics.Trace.Listeners`de seguimiento () durante su construcción.

Una implicación de esto es que si una aplicación tiene agentes de escucha de seguimiento asociados a la `System.Diagnostics.Trace` instancia predeterminada (por ejemplo, a `app.config` través de su archivo), las bibliotecas de cliente de Kusto emitirán seguimientos a esos agentes de escucha.

Este comportamiento se puede suprimir o controlar mediante programación o a través de un archivo de configuración.

## <a name="suppress-tracing-programmatically"></a>Suprimir seguimiento mediante programación

Para suprimir el seguimiento de las bibliotecas de cliente de Kusto mediante programación, invoque el siguiente fragmento de código antes de cargar la biblioteca pertinente:

```csharp
Kusto.Cloud.Platform.Utils.TraceSourceManager.SetTraceVerbosityForAll(
    Kusto.Cloud.Platform.Utils.TraceVerbosity.Fatal
    );
```

## <a name="suppressing-tracing-by-using-a-config-file"></a>Supresión del seguimiento mediante el uso de un archivo de configuración

Para suprimir el seguimiento de las bibliotecas de cliente de Kusto a través de un `Kusto.Cloud.Platform.dll.tweaks` archivo de configuración, modifique el `Kusto.Data` archivo (que se incluye con la biblioteca) de modo que el "retoque" adecuado ahora Lea:

```xml
    <!-- Overrides the default trace verbosity level -->
    <add key="Kusto.Cloud.Platform.Utils.Tracing.OverrideTraceVerbosityLevel" value="0" />
```

(Tenga en cuenta que para que el ajuste surta efecto, debe haber un signo menos en el valor `key`de).

Un medio alternativo es hacer lo siguiente:

```csharp
Kusto.Cloud.Platform.Utils.Anchor.Tweaks.SetProgrammaticAppSwitch(
    "Kusto.Cloud.Platform.Utils.Tracing.OverrideTraceVerbosityLevel",
    "0"
    );
```

## <a name="how-to-enable-the-kusto-client-libraries-tracing"></a>Cómo habilitar el seguimiento de las bibliotecas de cliente de Kusto

Para habilitar el seguimiento de las bibliotecas de cliente de Kusto, habilite el seguimiento de .NET en el archivo app. config de la aplicación. Por ejemplo, suponiendo que la aplicación `MyApp.exe` está usando la biblioteca de cliente de Kusto. Data. Después, al cambiar `MyApp.exe.config` el archivo para que incluya lo siguiente, se habilitará el seguimiento de Kusto. Data la próxima vez que se inicie la aplicación:

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

Esto configurará un agente de escucha de seguimiento que escribe en archivos CSV en un subdirectorio llamado `RollingLogs` ubicado en el directorio del proceso. (Por supuesto, cualquiera. También se puede usar la clase de escucha de seguimiento compatible con NET).

## <a name="how-to-enable-the-aad-client-libraries-adal-tracing"></a>Cómo habilitar el seguimiento de las bibliotecas de cliente de AAD (ADAL)

Una vez habilitada la traza de las bibliotecas de cliente de Kusto, es el seguimiento emitido por las bibliotecas de cliente de AAD (las bibliotecas de cliente de Kusto configuran automáticamente el seguimiento de ADAL).

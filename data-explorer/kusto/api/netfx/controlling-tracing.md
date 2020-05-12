---
title: 'Control y supresión del seguimiento del lado cliente de Kusto SDK: Azure Explorador de datos'
description: En este artículo se describe el control y la supresión del seguimiento del lado cliente del SDK de Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 10/23/2018
ms.openlocfilehash: 159036bbbe6e0415f56b36827b1913cba90fb705
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226183"
---
# <a name="controlling-and-suppressing-kusto-sdk-client-side-tracing"></a>Control y supresión del seguimiento del lado cliente del SDK de Kusto

Las bibliotecas de cliente de Kusto usan una plataforma común para el seguimiento. La plataforma usa un gran número de orígenes de seguimiento ( `System.Diagnostics.TraceSource` ) y cada uno está conectado al conjunto predeterminado de agentes de escucha de seguimiento ( `System.Diagnostics.Trace.Listeners` ) durante su construcción.

Si una aplicación tiene agentes de escucha de seguimiento asociados a la `System.Diagnostics.Trace` instancia predeterminada (por ejemplo, a través de su `app.config` archivo), las bibliotecas de cliente de Kusto emitirán seguimientos a esos agentes de escucha.

El seguimiento se puede suprimir o controlar mediante programación o a través de un archivo de configuración.

## <a name="suppress-tracing-programmatically"></a>Suprimir seguimiento mediante programación

Para suprimir el seguimiento de las bibliotecas de cliente de Kusto mediante programación, invoque este fragmento de código al cargar la biblioteca pertinente:

```csharp
Kusto.Cloud.Platform.Utils.TraceSourceManager.SetTraceVerbosityForAll(
    Kusto.Cloud.Platform.Utils.TraceVerbosity.Fatal
    );
```

## <a name="use-a-config-file-to-suppress-tracing"></a>Usar un archivo de configuración para suprimir el seguimiento 

Para suprimir el seguimiento de las bibliotecas de cliente de Kusto a través de un archivo de configuración, modifique el archivo `Kusto.Cloud.Platform.dll.tweaks` (que se incluye con la `Kusto.Data` biblioteca).

```xml
    <!-- Overrides the default trace verbosity level -->
    <add key="Kusto.Cloud.Platform.Utils.Tracing.OverrideTraceVerbosityLevel" value="0" />
```

> [!NOTE]
> Para que el ajuste surta efecto, no debe haber un signo menos en el valor de`key`

Una alternativa es:

```csharp
Kusto.Cloud.Platform.Utils.Anchor.Tweaks.SetProgrammaticAppSwitch(
    "Kusto.Cloud.Platform.Utils.Tracing.OverrideTraceVerbosityLevel",
    "0"
    );
```

## <a name="enable-the-kusto-client-libraries-tracing"></a>Habilitación del seguimiento de bibliotecas de cliente de Kusto

Para habilitar el seguimiento de las bibliotecas de cliente de Kusto, habilite el seguimiento de .NET en el *archivo app. config*de la aplicación. Por ejemplo, supongamos que la aplicación `MyApp.exe` utiliza la biblioteca de cliente Kusto. Data. Al cambiar el archivo *MyApp. exe. config* para incluir lo siguiente, se habilitará `Kusto.Data` el seguimiento la próxima vez que se inicie la aplicación.

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

El código configurará un agente de escucha de seguimiento que escriba en archivos CSV en un subdirectorio denominado *RollingLogs*. El subdirectorio se encuentra en el directorio del proceso.

> [!NOTE]
> Cualquier. También se puede usar la clase de escucha de seguimiento compatible con NET

## <a name="enable-the-azure-ad-client-libraries-adal-tracing"></a>Habilitar el seguimiento de las bibliotecas de cliente de Azure AD (ADAL)

Una vez habilitada la traza de las bibliotecas de cliente de Kusto, es el seguimiento de las bibliotecas de cliente de Azure AD. Las bibliotecas de cliente de Kusto configuran automáticamente el seguimiento de ADAL.

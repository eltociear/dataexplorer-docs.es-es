---
title: Solucionar problemas comunes en Kusto. Explorer
description: Obtenga información acerca de los problemas comunes de la instalación y ejecución de Kusto. Explorer y sus soluciones
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/13/2020
ms.openlocfilehash: aa3dec658ae3b817223c7946d55555cf6562cfb4
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83865989"
---
# <a name="troubleshooting"></a>Solución de problemas

En este documento se proporcionan dificultades comunes en la ejecución y el uso de Kusto. Explorer y se ofrecen soluciones. En este documento también se describe [cómo restablecer Kusto. Explorer](#reset-kustoexplorer).

## <a name="kustoexplorer-fails-to-start"></a>No se puede iniciar Kusto. Explorer

### <a name="kustoexplorer-shows-error-dialog-during-or-after-start-up"></a>Kusto. Explorer muestra un cuadro de diálogo de error durante o después del inicio

**Síntoma**

Al iniciarse, Kusto. Explorer muestra un `InvalidOperationException` error.

**Posible solución**

Este error puede sugerir que el sistema operativo se ha dañado o que faltan algunos de los módulos esenciales.
Para comprobar los archivos del sistema que faltan o están dañados, siga los pasos que se describen aquí:   
[https://support.microsoft.com/help/929833/use-the-system-file-checker-tool-to-repair-missing-or-corrupted-system](https://support.microsoft.com/help/929833/use-the-system-file-checker-tool-to-repair-missing-or-corrupted-system)

## <a name="kustoexplorer-always-downloads-even-when-there-are-no-updates"></a>Kusto. Explorer siempre se descarga, incluso cuando no hay ninguna actualización

**Síntoma**

Cada vez que abra Kusto. Explorer, se le pedirá que instale una nueva versión. Kusto. Explorer descarga todo el paquete, sin actualizar la versión ya instalada.

**Posible solución**

Este síntoma podría ser el resultado de daños en el almacén local de ClickOnce. Para borrar el almacén de ClickOnce local, ejecute el siguiente comando en un símbolo del sistema con privilegios elevados.

> [!Important]
> 1. Si hay otras instancias de aplicaciones ClickOnce o de `dfsvc.exe` , finalice antes de ejecutar este comando.
> 1. Todas las aplicaciones ClickOnce se reinstalarán automáticamente la próxima vez que se ejecuten, siempre y cuando tenga acceso a la ubicación de instalación original almacenada en el acceso directo de la aplicación. No se eliminarán los accesos directos de la aplicación.

```kusto
rd /q /s %userprofile%\appdata\local\apps\2.0
```

Intente volver a instalar Kusto. Explorer desde uno de los [reflejos de instalación](kusto-explorer.md#installing-kustoexplorer).

### <a name="clickonce-error-cannot-start-application"></a>Error de ClickOnce: no se puede iniciar la aplicación

**Síntomas**  

El programa no se inicia y muestra uno de los siguientes errores: 
* `External component has thrown an exception`
* `Value does not fall within the expected range`
* `The application binding data format is invalid.` 
* `Exception from HRESULT: 0x800736B2`

Puede explorar los detalles del error haciendo clic `Details` en el siguiente cuadro de diálogo de error:

![Error de ClickOnce](./Images/kusto-explorer-troubleshooting/clickonce-err-1.png)

```kusto
Following errors were detected during this operation.
    * System.ArgumentException
        - Value does not fall within the expected range.
        - Source: System.Deployment
        - Stack trace:
            at System.Deployment.Application.NativeMethods.CorLaunchApplication(UInt32 hostType, String applicationFullName, Int32 manifestPathsCount, String[] manifestPaths, Int32 activationDataCount, String[] activationData, PROCESS_INFORMATION processInformation)
            at System.Deployment.Application.ComponentStore.ActivateApplication(DefinitionAppId appId, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.SubscriptionStore.ActivateApplication(DefinitionAppId appId, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.ApplicationActivator.Activate(DefinitionAppId appId, AssemblyManifest appManifest, String activationParameter, Boolean useActivationParameter)
            at System.Deployment.Application.ApplicationActivator.ProcessOrFollowShortcut(String shortcutFile, String& errorPageUrl, TempFile& deployFile)
            at System.Deployment.Application.ApplicationActivator.PerformDeploymentActivation(Uri activationUri, Boolean isShortcut, String textualSubId, String deploymentProviderUrlFromExtension, BrowserSettings browserSettings, String& errorPageUrl)
            at System.Deployment.Application.ApplicationActivator.ActivateDeploymentWorker(Object state)
```

**Pasos de solución propuestos**

1. Desinstale Kusto. Explorer mediante `Programs and Features` ( `appwiz.cpl` ).

1. Intente ejecutar `CleanOnlineAppCache` y, a continuación, intente instalar de nuevo Kusto. Explorer. 
    Desde un símbolo del sistema con privilegios elevados: 
    
    ```kusto
    rundll32 %windir%\system32\dfshim.dll CleanOnlineAppCache
    ```

    Vuelva a instalar Kusto. Explorer desde uno de los [reflejos de instalación](kusto-explorer.md#installing-kustoexplorer).

1. Si la aplicación sigue sin iniciarse, elimine el almacén de ClickOnce local. Todas las aplicaciones ClickOnce se reinstalarán automáticamente la próxima vez que se ejecuten, siempre y cuando tenga acceso a la ubicación de instalación original almacenada en el acceso directo de la aplicación. No se eliminarán los accesos directos de la aplicación.

    Desde un símbolo del sistema con privilegios elevados:

    ```kusto
    rd /q /s %userprofile%\appdata\local\apps\2.0
    ```

    Vuelva a instalar Kusto. Explorer desde uno de los [reflejos de instalación](kusto-explorer.md#installing-kustoexplorer)

1. Si la aplicación sigue sin iniciarse:
    1. Quite los archivos de implementación temporales.
    1. Cambie el nombre de la carpeta local AppData de Kusto. Explorer.

        Desde un símbolo del sistema con privilegios elevados:

        ```kusto
        rd /s/q %userprofile%\AppData\Local\Temp\Deployment
        ren %LOCALAPPDATA%\Kusto.Explorer Kusto.Explorer.bak
        ```

    1. Vuelva a instalar Kusto. Explorer desde uno de los [reflejos de instalación](kusto-explorer.md#installing-kustoexplorer)

    1. Para restaurar las conexiones desde Kusto. Explorer. bak, desde un símbolo del sistema con privilegios elevados:

        ```kusto
        copy %LOCALAPPDATA%\Kusto.Explorer.bak\User*.xml %LOCALAPPDATA%\Kusto.Explorer
        ```

1. Si la aplicación sigue sin iniciarse:
    1. Habilite el registro detallado de ClickOnce mediante la creación de un valor de cadena de LogVerbosityLevel de 1 en:

        ```kusto
        HKEY_CURRENT_USER\Software\Classes\Software\Microsoft\Windows\CurrentVersion\Deployment
        ```
    
    1. Vuelva a realizar la reproducción.
    1. Envíe el resultado detallado a KEBugReport@microsoft.com . 

### <a name="clickonce-error-your-administrator-has-blocked-this-application-because-it-potentially-poses-a-security-risk-to-your-computer"></a>Error de ClickOnce: el administrador ha bloqueado esta aplicación porque puede suponer un riesgo de seguridad para el equipo

**Síntoma**  
La aplicación no se puede instalar con ninguno de los siguientes errores:
* `Your administrator has blocked this application because it potentially poses a security risk to your computer`.
* `Your security settings do not allow this application to be installed on your computer.`

**Solución**

Este síntoma podría deberse a que otra aplicación está invalidando el comportamiento predeterminado del mensaje de confianza de ClickOnce. 
1. Vea los valores de configuración predeterminados.
1. Compare las opciones de configuración con las de la máquina.
1. Restablezca las opciones de configuración según sea necesario, tal como se explica [en este artículo de procedimientos](https://docs.microsoft.com/visualstudio/deployment/how-to-configure-the-clickonce-trust-prompt-behavior).

### <a name="cleanup-application-data"></a>Limpiar datos de la aplicación

A veces, cuando los pasos anteriores de solución de problemas no ayudaron a iniciar Kusto. Explorer, la limpieza de los datos almacenados localmente puede ayudar.

Los datos almacenados por la aplicación Kusto. Explorer se pueden encontrar aquí: `C:\Users\\[your alias]\AppData\Local\Kusto.Explorer` .

> [!NOTE]
> La limpieza de los datos provocará una pérdida de pestañas abiertas (carpeta de recuperación), conexiones guardadas (carpeta conexiones) y configuración de la aplicación (carpeta UserSettings).

## <a name="reset-kustoexplorer"></a>Restablecer Kusto. Explorer

Si es necesario, puede restablecer completamente Kusto. Explorer. En el procedimiento siguiente se describe cómo restablecer Kusto. Explorer progresivamente, hasta que se Quite completamente del equipo y se deba instalar desde cero.

1. En Windows, Abra **cambiar o quitar un programa** (también conocido como **programas y características**).
1. Seleccione todos los elementos que empiecen por `Kusto.Explorer` .
1. Seleccione **Desinstalar**.

   Si en este procedimiento no se puede desinstalar la aplicación (un problema conocido con las aplicaciones ClickOnce), vea [este artículo de desbordamiento de pila, en el que se explica cómo hacerlo.

1. Elimine la carpeta `%LOCALAPPDATA%\Kusto.Explorer` , que quita todas las conexiones, el historial, etc.

1. Elimine la carpeta `%APPDATA%\Kusto` , que quita la caché de tokens de Kusto. Explorer. Deberá volver a autenticarse en todos los clústeres.

También es posible revertir a una versión específica de Kusto. Explorer:

1. Ejecute `appwiz.cpl`.
1. Seleccione **Kusto. Explorer** y seleccione **desinstalar o cambiar**.
1. Seleccione **restaurar la aplicación a su estado anterior**.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre la [interfaz de usuario de Kusto. Explorer](kusto-explorer.md#overview-of-the-user-interface)
* Más información sobre [la ejecución de Kusto. Explorer desde la línea de comandos](kusto-explorer-using.md#kustoexplorer-command-line-arguments)
* Más información sobre el [lenguaje de consulta de Kusto (KQL)](https://docs.microsoft.com/azure/kusto/query/)

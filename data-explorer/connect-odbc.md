---
title: Conexión a Azure Data Explorer con ODBC
description: En este artículo, aprenderá a configurar una conexión de conectividad abierta de bases de datos (ODBC) a Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/30/2019
ms.openlocfilehash: 156257613195cb53730273ac3a654908feac9601
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492728"
---
# <a name="connect-to-azure-data-explorer-with-odbc"></a>Conexión a Azure Data Explorer con ODBC

La conectividad abierta de bases de datos ([ODBC](/sql/odbc/reference/odbc-overview)) es una interfaz de programación de aplicaciones (API) ampliamente aceptada para el acceso a bases de datos. Use ODBC para conectarse a Azure Data Explorer desde las aplicaciones que no tienen un conector dedicado.

En segundo plano, las aplicaciones llaman a las funciones en la interfaz de ODBC, las que se implementan en módulos específicos para la base de datos llamados *controladores*. Azure Data Explorer admite un subconjunto del protocolo de comunicación SQL Server ([MS-TDS](kusto/api/tds/index.md)) para poder usar el controlador ODBC para SQL Server.

Con el vídeo siguiente aprenderá a crear una conexión de ODBC. 

> [!VIDEO https://www.youtube.com/embed/qA5wxhrOwog]

También puede [configurar el origen de datos ODBC](#configure-the-odbc-data-source) como se detalla a continuación. 

En este artículo, aprenderá a usar el controlador ODBC para SQL Server, de manera que pueda conectarse a Azure Data Explorer desde cualquier aplicación que admita ODBC. 

## <a name="prerequisites"></a>Prerequisites

Necesita lo siguiente:

* [Microsoft ODBC Driver for SQL Server, versión 17.2.0.1 o posterior](/sql/connect/odbc/download-odbc-driver-for-sql-server) para el sistema operativo.

## <a name="configure-the-odbc-data-source"></a>Configuración del origen de datos ODBC

Siga estos pasos para configurar un origen de datos ODBC con el controlador ODBC para SQL Server.

1. En Windows, busque *Orígenes de datos ODBC* y abra la aplicación de escritorio Orígenes de datos ODBC.

1. Seleccione **Agregar**.

    ![Agregar origen de datos](media/connect-odbc/add-data-source.png)

1. Seleccione **Controlador ODBC 17 para SQL Server** y, luego, **Finalizar**.

    ![Selección del controlador](media/connect-odbc/select-driver.png)

1. Escriba un nombre y una descripción para la conexión y el clúster al que quiere conectarse y, luego, seleccione **Siguiente**. La dirección URL del clúster debe tener el formato *\<NombreDeClúster\>.\<Región\>.kusto.windows.net*.

    ![Selección del servidor](media/connect-odbc/select-server.png)

1. Seleccione **Integrado en Active Directory** y, luego, **Siguiente**.

    ![Integrado en Active Directory](media/connect-odbc/active-directory-integrated.png)

1. Seleccione la base de datos con los datos de ejemplo y, luego, **Siguiente**.

    ![Cambio de la base de datos predeterminada](media/connect-odbc/change-default-database.png)

1. En la pantalla siguiente, deje los valores predeterminados en todas las opciones y, luego, seleccione **Finalizar**.

1. Seleccione **Probar origen de datos**.

    ![Prueba del origen de datos](media/connect-odbc/test-data-source.png)

1. Compruebe que la prueba se realizó correctamente y, luego, haga clic en **Aceptar**. Si la prueba no se realizó correctamente, revise los valores que especificó en los pasos anteriores y asegúrese de tener los permisos suficientes para conectarse al clúster.

    ![La prueba se realizó correctamente](media/connect-odbc/test-succeeded.png)

## <a name="next-steps"></a>Pasos siguientes

* [Connect to Azure Data Explorer from Tableau](tableau.md) (Conexión a Azure Data Explorer desde Tableau)
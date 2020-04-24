---
title: Visualización de datos mediante una consulta de Kusto de Azure Data Explorer importada en Microsoft Excel
description: En este artículo, aprenderá a importar una consulta de Kusto de Azure Data Explorer en Microsoft Excel.
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/30/2019
ms.openlocfilehash: 4999000e2084922b43b8085034f545d4b5c644a9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493624"
---
# <a name="visualize-data-using-an-azure-data-explorer-kusto-query-imported-into-microsoft-excel"></a>Visualización de datos mediante una consulta de Kusto de Azure Data Explorer importada en Microsoft Excel

Azure Data Explorer proporciona dos opciones para conectarse a datos de Excel: usar el conector nativo o importar una consulta desde Azure Data Explorer. En este artículo se muestra cómo importar una consulta de Azure Data Explorer a Excel para visualizar datos. Agregue la consulta de Kusto como origen de datos de Excel para realizar visualizaciones o cálculos adicionales en los datos.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Una cuenta de correo electrónico organizativa que sea miembro de Azure Active Directory, para que pueda conectarse al [clúster de ayuda de Azure Data Explorer](https://dataexplorer.azure.com/clusters/help/databases/Samples) 
.<br>or</br>
* Cree [una base de datos y un clúster de prueba](create-cluster-database-portal.md) e inicie sesión en la [aplicación de interfaz de usuario web de Azure Data Explorer](https://dataexplorer.azure.com/).

## <a name="define-kusto-query-as-an-excel-data-source"></a>Definición de la consulta de Kusto como origen de datos de Excel

1. En la [interfaz de usuario web de Azure Data Explorer](https://dataexplorer.azure.com/clusters/help/databases/Samples), ejecute la consulta y compruebe los resultados.

1. Seleccione la pestaña **Share** (Compartir) y, luego, **Query to Power BI** (Consultar en Power BI).

    ![Consulta de interfaz de usuario web en Power BI](media/excel-blank-query/web-ui-query-to-powerbi.png)

1. Aparece una ventana con la siguiente notificación:

    ![exportar consulta al portapapeles](media/excel-blank-query/query-exported-to-clipboard.png)

1. Abra **Microsoft Excel**.

1. En la pestaña **Datos**, seleccione **Obtener datos** > **De otras fuentes** > **Consulta en blanco**.

    ![Obtención de datos y selección de la consulta en blanco](media/excel-blank-query/get-data-blank-query.png)

1. Se abre la ventana **Editor de Power Query**. En esta ventana, seleccione **Editor avanzado**.

    ![Ventana del editor de Power Query](media/excel-blank-query/power-query-editor.png)

1. En la ventana **Editor avanzado**, pegue la consulta que exportó al Portapapeles y, luego, seleccione **Listo**.

    ![Consulta del Editor avanzado](media/excel-blank-query/advanced-editor-query.png)    

1. Para realizar la autenticación, seleccione **Editar credenciales**.

    ![Editar credenciales](media/excel-blank-query/edit-credentials.png)

1. Seleccione **Cuenta organizativa** e **Iniciar sesión**. Complete el proceso de inicio de sesión y seleccione **Conectar**.

    ![Inicio de sesión completo](media/excel-blank-query/complete-sign-in.png)

    Repita los pasos anteriores para agregar más consultas. Puede cambiar el nombre de las consultas por otros más significativos.

1. Seleccione el botón **Close & Load** (Cerrar y cargar) para obtener los datos en Excel.

    ![Selección de Close & Load (Cerrar y cargar)](media/excel-blank-query/close-and-load.png)

1. Ahora los datos están en Excel. Use el botón **Actualizar** para actualizar la lista.

    ![Visualización de los datos en Excel](media/excel-blank-query/data-in-excel.png)

## <a name="next-steps"></a>Pasos siguientes

[Visualización de datos mediante el conector de Azure Data Explorer para Excel](excel-connector.md)
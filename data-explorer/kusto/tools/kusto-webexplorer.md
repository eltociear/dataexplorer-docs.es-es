---
title: Kusto.WebExplorer - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Kusto.WebExplorer en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 1f6926df09a207cfea2b9201ef57f36932a63f74
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523875"
---
# <a name="kustowebexplorer"></a>Kusto.WebExplorer

Kusto.WebExplorer es una aplicación web que se puede utilizar para enviar consultas y controlar comandos a un servicio Kusto. La aplicación sehttps://dataexplorer.azure.com/aloja en [ ]https://aka.ms/kwey enlaza con short por [ ].



Kusto.WebExplorer también puede ser alojado por otros portales web en un IFRAME HTML.
(Por ejemplo, esto lo hace [Azure Portal](https://portal.azure.com).) Consulte [el IDE](../api/monaco/monaco-kusto.md) de Mónaco para obtener más información sobre cómo alojarlo y el editor de Mónaco que utiliza.

## <a name="connect-to-multiple-clusters"></a>Conéctese a varios clústeres

Ahora puede conectar varios clústeres y cambiar entre bases de datos y clústeres.
La herramienta está diseñada para identificar fácilmente el clúster y la base de datos a la que está conectado.

![texto alternativo](./Images/KustoTools-WebExplorer/AddingCluster.gif "AddingCluster")

## <a name="recall-results"></a>Recuperar resultados

A menudo, durante el análisis, ejecutamos varias consultas y es posible que tengamos que volver a visitar los resultados de las consultas anteriores. Puede utilizar esta característica para recuperar los resultados sin tener que volver a ejecutar la consulta. Los datos se sirven desde la memoria caché del lado cliente local.

![texto alternativo](./Images/KustoTools-WebExplorer/RecallResults.gif "RecallResults")

## <a name="enhanced-results-grid-control"></a>Control mejorado de la cuadrícula de resultados

La cuadrícula de tablas le permite seleccionar varias filas, columnas y celdas. Calcular agregados seleccionando varias celdas (como Excel) y pivotar los datos.

![texto alternativo](./Images/KustoTools-WebExplorer/EnhancedGrid.gif "EnhancedGrid")

## <a name="intellisense--formatting"></a>Formato de & Intellisense

Puede utilizar el formato de impresión bonita mediante la tecla de método abreviado "Mayús + Alt + F", el plegado de código (esquematización) e IntelliSense.

![texto alternativo](./Images/KustoTools-WebExplorer/Formating.gif "Formato")

## <a name="deep-linking"></a>Enlace profundo

Puede copiar solo el vínculo profundo o el vínculo profundo y la consulta. También puede dar formato a la dirección URL para incluir el clúster, la base de datos y la consulta mediante la siguiente plantilla:

`https://dataexplorer.azure.com/`[`clusters/` Clúster`/databases/` [`?` Base *de* *datos* [ *Opciones*]]]

Se pueden especificar las siguientes opciones:

* `workspace=empty`: indica que se va a crear un nuevo espacio de trabajo vacío (no se realizará ninguna recuperación de clústeres, pestañas y consultas anteriores).



![texto alternativo](./Images/KustoTools-WebExplorer/DeepLink.gif "DeepLink")

## <a name="feedback"></a>Comentarios

Puede enviar sus comentarios a través de la herramienta.
![texto alternativo](./Images/KustoTools-WebExplorer/Feedback.gif "Comentarios")
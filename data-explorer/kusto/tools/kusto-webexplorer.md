---
title: 'Kusto. webexplorer: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe Kusto. webexplorer en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: d53f12c4a0c4dd2bce669dbe004b8f325db27af5
ms.sourcegitcommit: 4986354cc1ba25c584e4f3c7eac7b5ff499f0cf1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84856384"
---
# <a name="kustowebexplorer"></a>Kusto. webexplorer

Kusto. webexplorer es una aplicación web que se puede usar para enviar consultas y comandos de control a un servicio de Kusto. La aplicación se hospeda en https://dataexplorer.azure.com/ y está vinculada en breve por https://aka.ms/kwe .



Kusto. webexplorer también se puede hospedar en otros portales web en un IFRAME de HTML.
(Por ejemplo, esto se realiza mediante el [Azure portal](https://portal.azure.com)). Consulte el [IDE de Mónaco](../api/monaco/monaco-kusto.md) para obtener más información sobre cómo hospedarlo y el editor de Mónaco que usa.

## <a name="connect-to-multiple-clusters"></a>Conexión a varios clústeres

Ahora puede conectar varios clústeres y cambiar entre bases de datos y clústeres.
La herramienta está diseñada para identificar fácilmente el clúster y la base de datos a los que está conectado.

![texto alternativo](./Images/KustoTools-WebExplorer/AddingCluster.gif "AddingCluster")

## <a name="recall-results"></a>Resultados de la recuperación

A menudo durante el análisis, se ejecutan varias consultas y es posible que tenga que volver a revisar los resultados de las consultas anteriores. Puede usar esta característica para recuperar los resultados sin tener que volver a ejecutar la consulta. Los datos se sirven desde la memoria caché del lado cliente local.

![texto alternativo](./Images/KustoTools-WebExplorer/RecallResults.gif "RecallResults")

## <a name="enhanced-results-grid-control"></a>Control de cuadrícula de resultados mejorado

La cuadrícula de tabla permite seleccionar varias filas, columnas y celdas. Calcule los agregados seleccionando varias celdas (como Excel) y dinamizando los datos.

![texto alternativo](./Images/KustoTools-WebExplorer/EnhancedGrid.gif "EnhancedGrid")

## <a name="intellisense--formatting"></a>Formato de & de IntelliSense

Puede usar el formato de impresión con sangría mediante la tecla de método abreviado "Mayús + Alt + F", el plegado de código (esquematización) e IntelliSense.

![texto alternativo](./Images/KustoTools-WebExplorer/Formating.gif "Dar formato")

## <a name="deep-linking"></a>Vinculación profunda

Puede copiar solo el vínculo profundo o profundo y la consulta. También puede dar formato a la dirección URL para incluir el clúster, la base de datos y la consulta mediante la siguiente plantilla:

`https://dataexplorer.azure.com/`[ `clusters/` *Cluster* [ `/databases/` *Database* [ `?` *Opciones*]]]

Se pueden especificar las siguientes opciones:

* `workspace=empty`: Indica que se va a crear un nuevo área de trabajo vacía (no se realizará ninguna recuperación de los clústeres anteriores, las pestañas y las consultas).



![texto alternativo](./Images/KustoTools-WebExplorer/DeepLink.gif "DeepLink")

## <a name="feedback"></a>Comentarios

Puede enviar sus comentarios a través de la herramienta.
![texto alternativo](./Images/KustoTools-WebExplorer/Feedback.gif "Comentarios")

---
title: Kusto WPA - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Kusto WPA en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: 2d8c5a9b11d8045897d8a9bd798e40ceb0f48b6a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81503305"
---
# <a name="kusto-wpa"></a>Kusto WPA

Kusto WPA agrega la capacidad de ejecutar y visualizar los resultados de la consulta Kusto en Windows Performance Analyzer (WPA). Consta de dos partes principales:

1. Una herramienta de iniciador que puede aceptar consultas `Share` &gt; `Query to Clipboard`compartidas de`.kpq`Kusto.Explorer (a través) y generar archivos de consulta ( ) para ser procesados por WPA.

1. Un plugin WPA que puede "abrir"`.kpq`los archivos de consulta generados ( ). Al abrir un archivo de este tipo, se envía automáticamente la consulta especificada en el archivo a un clúster de Kusto y se cargan los resultados como si provinieran de un archivo ETL "estándar".

Consultehttps://aka.ms/kustowpa[ ] para obtener más información y descargar.
---
title: Refactorización de código de Kusto Explorer - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la refactorización de código de Kusto Explorer en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/05/2019
ms.openlocfilehash: 0a89cf9c648fc4811d56c22012cdb25d5505eb4c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523994"
---
# <a name="kusto-explorer-code-refactoring"></a>Refactorización de código de Kusto Explorer

Al igual que otros IDE, Kusto.Explorer ofrece varias características para la edición y refactorización de consultas KQL.

## <a name="rename-variable-or-column-name"></a>Cambiar el nombre de la variable o columna

Al `Ctrl` + `R`hacer `Ctrl` + `R` clic en , en la ventana Editor de consultas le permitirá cambiar el nombre del símbolo seleccionado actualmente.

Vea la siguiente instantánea que muestra la experiencia:

![texto alternativo](./Images/KustoTools-KustoExplorer/ke-refactor-rename.gif "refactorizar el nombre")

## <a name="extract-scalars-as-let-expressions"></a>Extraer escalares como `let` expresiones

Puede promocionar literales `let` seleccionados actualmente como expresión haciendo `Alt` + `Ctrl` + `M`clic en . 

![texto alternativo](./Images/KustoTools-KustoExplorer/ke-extract-as-let-literal.gif "extracto-as-let-literal")

## <a name="extract-tabular-statements-as-let-expressions"></a>Extraer instrucciones tabulares como `let` expresiones

También puede promover expresiones tabulares como `let` instrucciones seleccionando su texto y, a continuación, haciendo `Alt` + `Ctrl` + `M`clic en . 

![texto alternativo](./Images/KustoTools-KustoExplorer/ke-extract-as-let-tabular.gif "extracto como-let-tabular")

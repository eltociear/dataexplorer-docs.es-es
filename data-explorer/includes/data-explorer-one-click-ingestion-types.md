---
author: orspod
ms.service: data-explorer
ms.topic: include
ms.date: 30/03/2020
ms.author: orspodek
ms.openlocfilehash: 56095df921864896dacdb100302d686d66f40572
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82109614"
---
## <a name="select-an-ingestion-type"></a>Selección de un tipo de ingesta

En **Ingestion type** (Tipo de ingesta), seleccione una de las opciones siguientes:
   * **from storage** (de almacenamiento): en el campo **Lin to storage** (Vínculo al almacenamiento), agregue la dirección URL de la cuenta de almacenamiento. Use una [dirección URL de SAS de blob](/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container) para las cuentas de almacenamiento privadas.
   
      ![Ingesta con un clic desde el almacenamiento](media/data-explorer-one-click-ingestion-types/from-storage-blob.png)

    * **from file** (De archivo): seleccione **Browse** (Examinar) para buscar el archivo o bien arrastre el archivo al campo.
  
      ![Ingesta con un clic desde el archivo](media/data-explorer-one-click-ingestion-types/from-file.png)

    * **from container** (De contenedor): en el campo **Link to storage** (Vínculo al almacenamiento), agregue la [URL de SAS](/azure/vs-azure-tools-storage-explorer-blobs#get-the-sas-for-a-blob-container) del contenedor y, opcionalmente, especifique el tamaño del ejemplo.

      ![Ingesta con un clic desde el contenedor](media/data-explorer-one-click-ingestion-types/from-container.png)

  Aparece un ejemplo de los datos. Si lo desea, puede filtrarla para mostrar solo los archivos que empiezan por caracteres específicos. Al ajustar los filtros, la vista previa se actualiza automáticamente.
  
  Por ejemplo, puede filtrar por todos los archivos que comienzan por la palabra *data* y finalizar con una extensión *.csv.gz*.

  ![Filtro de ingesta con un clic](media/data-explorer-one-click-ingestion-types/from-container-with-filter.png)

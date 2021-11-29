---
lab:
  title: Abilitare i provider di risorse
  module: Setup
ms.openlocfilehash: 5de107bd550a03696f2c1e6fc6bfb9e4a7bdcc12
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625955"
---
# <a name="enable-resource-providers"></a>Abilitare i provider di risorse

Alcuni provider di risorse devono essere registrati nella sottoscrizione di Azure. Seguire questa procedura per assicurarsi che siano registrati.

1. Accedere al portale di Azure all'indirizzo `https://portal.azure.com` usando le credenziali Microsoft associate alla sottoscrizione.
2. Nella **home page** selezionare **Sottoscrizioni** oppure espandere il menu **&#8801;** , selezionare **Tutti i servizi** e quindi, nella categoria **Tutti**, selezionare **Sottoscrizioni**.
3. Selezionare la sottoscrizione di Azure(se si hanno più sottoscrizioni, selezionare quella creata riscattando Azure Pass).
4. Nel pannello della sottoscrizione, nella sezione **Impostazioni** del riquadro sinistro, selezionare **Provider di risorse**.
5. Nell'elenco dei provider di risorse verificare che i provider seguenti siano registrati (in caso contrario, selezionarli e fare clic su **Registra**):
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance

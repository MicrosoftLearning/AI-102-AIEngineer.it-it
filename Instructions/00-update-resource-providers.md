---
lab:
  title: Abilitare i provider di risorse
  module: Setup
ms.openlocfilehash: 2cd48d0d9f67b9bab3e64452bf6199e1f438492a
ms.sourcegitcommit: e06fbeaa3bc15e4dbe99f72216dfeeb27f58babd
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 02/15/2022
ms.locfileid: "138765492"
---
# <a name="enable-resource-providers"></a>Abilitare i provider di risorse

Alcuni provider di risorse devono essere registrati nella sottoscrizione di Azure. Seguire questa procedura per assicurarsi che siano registrati.

1. Accedere al portale di Azure all'indirizzo `https://portal.azure.com` usando le credenziali Microsoft associate alla sottoscrizione.
2. Nella **home page** selezionare **Sottoscrizioni** oppure espandere il menu **&#8801;** , selezionare **Tutti i servizi** e quindi, nella categoria **Generale**, selezionare **Sottoscrizioni**.
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

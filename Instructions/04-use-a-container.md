---
lab:
  title: Usare un contenitore di Servizi cognitivi
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: c222f5526a09ee1ae2aad3732fe2e29eb14c12ef
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801341"
---
# <a name="use-a-cognitive-services-container"></a>Usare un contenitore di Servizi cognitivi

L'uso dei servizi cognitivi ospitati in Azure consente agli sviluppatori di applicazioni di concentrarsi sull'infrastruttura per il proprio codice, sfruttando i servizi scalabili gestiti da Microsoft. In molti scenari, però, le organizzazioni richiedono un maggiore controllo sull'infrastruttura del servizio e sui dati passati tra i servizi.

Molte API dei servizi cognitivi possono essere in pacchetto e distribuite in un *contenitore*, consentendo alle organizzazioni di ospitare i servizi cognitivi nella propria infrastruttura. ad esempio nei server Docker locali, Istanze di Azure Container o cluster di servizi Azure Kubernetes. I servizi cognitivi in contenitori devono comunicare con un account servizi cognitivi basato su Azure per supportare la fatturazione, ma i dati dell'applicazione non vengono passati al servizio back-end e le organizzazioni hanno un maggiore controllo sulla configurazione della distribuzione dei contenitori, consentendo soluzioni personalizzate per l'autenticazione, la scalabilità e altre considerazioni.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se è già stato clonato il repository di codice **AI-102-AIEngineer** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="provision-a-cognitive-services-resource"></a>Effettuare il provisioning di una risorsa di Servizi cognitivi

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Servizi cognitivi**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Servizi cognitivi* e creare una risorsa di **Servizi cognitivi** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

## <a name="deploy-and-run-a-text-analytics-container"></a>Distribuire ed eseguire un contenitore Analisi del testo

Molte API dei servizi cognitivi di uso comune sono disponibili nelle immagini del contenitore. Per un elenco completo, vedere la [documentazione di Servizi cognitivi](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#container-availability-in-azure-cognitive-services). In questo esercizio si userà l'immagine del contenitore per l'API di *rilevamento lingua* di Analisi del testo, ma i principi sono gli stessi per tutte le immagini disponibili.

1. Nella pagina **Home** del portale di Azure selezionare il pulsante **&#65291;Crea una risorsa**, cercare *istanze di contenitore* e creare una risorsa **Istanze di Container** con le impostazioni seguenti:

    - **Nozioni di base**:
        - **Sottoscrizione**: *la propria sottoscrizione di Azure*
        - **Gruppo di risorse**: *scegliere il gruppo di risorse contenente la risorsa di Servizi cognitivi*
        - **Nome contenitore**: *immettere un nome univoco*
        - **Area**: *scegliere una qualsiasi area disponibile*
        - **Origine immagine**: Docker Hub o altro registro
        - **Tipo di immagine**: Pubblico
        - **Immagine**: `mcr.microsoft.com/azure-cognitive-services/textanalytics/language:1.1.013570001-amd64`
        - **Tipo di sistema operativo**: Linux
        - **Dimensioni**: 1 vCPU, 4 GB di memoria
    - **Rete**:
        - **Tipo di rete**: Pubblico
        - **Etichetta del nome DNS**: *immettere un nome univoco per l'endpoint contenitore*
        - **Porte**: *cambiare la porta TCP da 80 a **5000***
    - **Avanzate**:
        - **Criteri di riavvio**: In caso di esito negativo
        - **Variabili di ambiente**:

            | Contrassegna come sicura | Chiave | Valore |
            | -------------- | --- | ----- |
            | Sì | `ApiKey` | *Chiave per la risorsa di Servizi cognitivi* |
            | Sì | `Billing` | *URI dell'endpoint per la risorsa di Servizi cognitivi* |
            | No | `Eula` | `accept` |

        - **Override comando**: [ ]
    - **Tag**:
        - *Non aggiungere tag*

2. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
3. Osservare le proprietà seguenti della risorsa dell'istanza di contenitore nella pagina **Panoramica**:
    - **Stato**: deve essere *In esecuzione*.
    - **Indirizzo IP**: indirizzo IP pubblico che è possibile usare per accedere alle istanze di contenitore.
    - **FQDN**: *nome di dominio completo* della risorsa delle istanze di contenitore. È possibile usarlo in alternativa all'indirizzo IP per accedere alle istanze di contenitore.

    > **Nota**: in questo esercizio è stata distribuita l'immagine del contenitore di Servizi cognitivi per la traduzione testuale in una risorsa di Istanze di Azure Container. È possibile usare un approccio simile per distribuirlo in un host *[Docker](https://www.docker.com/products/docker-desktop)* nel computer o nella rete eseguendo il comando seguente (su una singola riga) per distribuire il contenitore di rilevamento lingua nell'istanza di Docker locale, sostituendo *&lt;yourEndpoint&gt;* e *&lt;yourKey&gt;* con l'URI dell'endpoint e una delle chiavi per la risorsa di Servizi cognitivi.
    >
    > ```
    > docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 mcr.microsoft.com/azure-cognitive-services/textanalytics/language Eula=accept Billing=<yourEndpoint> ApiKey=<yourKey>
    > ```
    >
    > Il comando cerca l'immagine nel computer locale e, se non la trova, la estrae dal registro immagini *mcr.microsoft.com* e la distribuisce nell'istanza di Docker. Al termine della distribuzione, il contenitore verrà avviato e sarà in ascolto delle richieste in ingresso sulla porta 5000.

## <a name="use-the-container"></a>Usare il contenitore

1. Nella cartella **04-containers** di Visual Studio Code aprire **rest-test.cmd** e modificare il comando **curl** in essa contenuto (mostrato di seguito), sostituendo *&lt;your_ACI_IP_address_or_FQDN&gt;* con l'indirizzo IP o il nome di dominio completo (FQDN) del contenitore.

    ```
    curl -X POST "http://<your_ACI_IP_address_or_FQDN>:5000/text/analytics/v3.0/languages?" -H "Content-Type: application/json" --data-ascii "{'documents':[{'id':1,'text':'Hello world.'},{'id':2,'text':'Salut tout le monde.'}]}"
    ```

2. Salvare le modifiche apportate allo script. Tenere presente che non è necessario specificare l'endpoint o la chiave di Servizi cognitivi. La richiesta viene elaborata dal servizio in contenitori. A sua volta, il contenitore comunica periodicamente con il servizio in Azure per segnalare l'utilizzo ai fini della fatturazione, ma non invia i dati della richiesta.
3. Fare clic con il pulsante destro del mouse sulla cartella **04-containers** e aprire un terminale integrato. Immettere quindi il comando seguente per eseguire lo script:

    ```
    rest-test
    ```

4. Verificare che il comando restituisca un documento JSON contenente informazioni sulla lingua rilevata nei due documenti di input (che devono essere inglese e francese).

## <a name="clean-up"></a>Eseguire la pulizia

Una volta completati gli esperimenti con l'istanza di contenitore, è consigliabile eliminarla.

1. Nel portale di Azure aprire il gruppo di risorse in cui sono state create le risorse per questo esercizio.
2. Selezionare la risorsa dell'istanza di contenitore ed eliminarla.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla containerizzazione di Servizi cognitivi, vedere la [documentazione di Servizi cognitivi](https://docs.microsoft.com/azure/cognitive-services/containers/).

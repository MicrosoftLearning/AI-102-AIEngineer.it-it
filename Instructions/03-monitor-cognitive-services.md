---
lab:
  title: Monitorare Servizi cognitivi
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: caf885516acab74cff46d3a4f807ee98aed446a3
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625821"
---
# <a name="monitor-cognitive-services"></a>Monitorare Servizi cognitivi

Servizi cognitivi di Azure può essere una parte essenziale di un'infrastruttura generale delle applicazioni. È importante essere in grado di monitorare l'attività e ricevere avvisi in caso di problemi che potrebbero richiedere attenzione.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se è già stato clonato il repository di codice **AI-102-AIEngineer** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.
2. Aprire il pannello (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale. Non è importante usare una cartella specifica.
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
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Prendere nota dell'URI dell'endpoint, che sarà necessario più avanti.

## <a name="configure-an-alert"></a>Configurare un avviso

Iniziamo il monitoraggio definendo una regola di avviso in modo da poter rilevare l'attività nella risorsa di Servizi cognitivi.

1. Nel portale di Azure passare alla risorsa di Servizi cognitivi e visualizzare la relativa pagina **Avvisi** (nella sezione **Monitoraggio**).
2. Selezionare **+ Nuova regola di avviso**
3. Nella pagina **Crea regola di avviso** in **Ambito** verificare che la risorsa di Servizi cognitivi sia elencata.
4. In **Condizione** fare clic su **Aggiungi condizione**. A destra viene visualizzato il riquadro **Configura logica dei segnali** in cui è possibile selezionare un tipo di segnale da monitorare.
5. Nell'elenco **tipo di segnale** selezionare **Log attività** e quindi nell'elenco filtrato selezionare **Elenca chiavi**.
6. Esaminare l'attività nelle ultime 6 ore e quindi selezionare **Fine**.
7. Nella pagina **Crea regola di avviso** in **Azioni** si noti che è possibile specificare un *gruppo di azioni*. In questo modo è possibile configurare azioni automatiche quando viene generato un avviso, ad esempio l'invio di una notifica tramite posta elettronica. Questa operazione non verrà eseguita in questo esercizio, ma può essere utile eseguirla in un ambiente di produzione.
8. Nella sezione **Alert Rules Details** (Dettagli regole di avviso) impostare il **Nome regola di avviso** su **Key List Alert** (Avviso elenco chiavi) e fare clic su **Crea regola di avviso**. Attendere la creazione della regola di avviso.
9. In Visual Studio Code fare clic con il pulsante destro del mouse sulla cartella **03-monitor** e aprire un terminale integrato. Immettere quindi il comando seguente per accedere alla sottoscrizione di Azure usando l'interfaccia della riga di comando di Azure.

    ```
    az login
    ```

    Verrà aperta una scheda del Web browser in cui si richiede di accedere ad Azure. Accedere, quindi chiudere la scheda del browser e tornare a Visual Studio Code.

    > **Suggerimento**: se si hanno più sottoscrizioni, è necessario assicurarsi di usare quella che contiene la risorsa di Servizi cognitivi.  Usare questo comando per determinare la sottoscrizione corrente.
    >
    > ```
    > az account show
    > ```
    >
    > Se è necessario modificare la sottoscrizione, eseguire questo comando, modificando *&lt;subscriptionName&gt;* con il nome della sottoscrizione corretto.
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. È ora possibile usare il comando seguente per ottenere l'elenco delle chiavi di Servizi cognitivi, sostituendo *&lt;resourceName&gt;* con il nome della risorsa di Servizi cognitivi e *&lt;resourceGroup&gt;* con il nome del gruppo di risorse in cui è stata creata.

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

Il comando restituisce un elenco delle chiavi per la risorsa di Servizi cognitivi.

11. Tornare al browser contenente il portale di Azure e aggiornare la **pagina Avviso**. Nella tabella verrà visualizzato un avviso di **gravità 4** (in caso contrario, attendere fino a cinque minuti e aggiornare di nuovo).
12. Selezionare l'avviso per visualizzarne i dettagli.

## <a name="visualize-a-metric"></a>Visualizzare una metrica

Oltre a definire gli avvisi, è possibile visualizzare le metriche della risorsa di Servizi cognitivi per monitorarne l'utilizzo.

1. Nella pagina della risorsa di Servizi cognitivi nel portale di Azure selezionare **Metriche** (nella sezione **Monitoraggio**).
2. Se non è presente alcun grafico esistente, selezionare **+ Nuovo grafico**. Quindi, nell'elenco **Metrica** esaminare le metriche che è possibile visualizzare e selezionare **Chiamate totali**.
3. Nell'elenco **Aggregazione**, selezionare **Conteggio**.  Questo consentirà di monitorare le chiamate totali alla risorsa di Servizi cognitivi, un'opzione utile per determinare il livello d'uso del servizio in un periodo di tempo.
4. Per generare alcune richieste al servizio cognitivo, si userà **curl**, uno strumento da riga di comando per le richieste HTTP. Nella cartella **03-monitor** di Visual Studio Code aprire **rest-test.cmd** e modificare il comando **curl** in esso contenuto (illustrato di seguito), sostituendo *&lt;yourEndpoint&gt;* e *&lt;yourKey&gt;* con l'URI dell'endpoint e la chiave **Key1** per usare l'API Analisi del testo nella risorsa di Servizi cognitivi.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. Salvare le modifiche e quindi nel terminale integrato per la cartella **03-monitor** eseguire il comando seguente:

    ```
    rest-test
    ```

Il comando restituisce un documento JSON contenente informazioni sulla lingua rilevata nei dati di input (che sarà l'inglese).

6. Eseguire nuovamente il comando **rest-test** più volte per generare alcune attività di chiamata (è possibile usare il tasto **^** per scorrere i comandi precedenti).
7. Tornare alla pagina **Metriche** nel portale di Azure e aggiornare il grafico del conteggio **Chiamate totali**. Potrebbero essere necessari alcuni minuti prima che le chiamate effettuate usando *curl* si riflettano nel grafico. Continuare ad aggiornare il grafico fino a quando non viene aggiornato includendole.

## <a name="more-information"></a>Altre informazioni

Una delle opzioni per il monitoraggio dei servizi cognitivi consiste nell'usare la *registrazione diagnostica*. Una volta abilitata, la registrazione diagnostica acquisisce informazioni dettagliate sull'utilizzo delle risorse di Servizi cognitivi e può essere un utile strumento di monitoraggio e debug. La generazione di informazioni può richiedere più di un'ora dopo aver configurato la registrazione diagnostica, motivo per cui non è stata esaminata in questo esercizio, ma è possibile ottenere altre informazioni su tale operazione nella [documentazione di Servizi cognitivi](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging).

---
lab:
  title: Introduzione a Servizi cognitivi
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 184092faafa5e4a138bd9d9cc07f253110f7d8c3
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625829"
---
# <a name="get-started-with-cognitive-services"></a>Introduzione a Servizi cognitivi

In questo esercizio si inizieranno a usare i servizi cognitivi creando una risorsa di **Servizi cognitivi** nella sottoscrizione e usandola da un'applicazione client. L'obiettivo dell'esercizio non è sviluppare competenze in un particolare servizio, ma piuttosto acquisire familiarità con un modello generale per il provisioning e l'uso di servizi cognitivi come sviluppatore.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="provision-a-cognitive-services-resource"></a>Effettuare il provisioning di una risorsa di Servizi cognitivi

I servizi basati sul cloud di Servizi cognitivi di Azure incapsulano funzionalità di intelligenza artificiale che è possibile incorporare nelle applicazioni. È possibile effettuare il provisioning di singole risorse di Servizi cognitivi per API specifiche (ad esempio **Analisi del testo** o **Visione artificiale**) oppure effettuare il provisioning di una risorsa generale di **Servizi cognitivi** che fornisce l'accesso a più API di Servizi cognitivi tramite un singolo endpoint e una chiave. In questo caso, si userà una singola risorsa di **Servizi cognitivi**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Servizi cognitivi* e creare una risorsa di **Servizi cognitivi** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
5. Passare alla risorsa e visualizzare la relativa pagina **Chiavi ed endpoint**. Questa pagina contiene le informazioni necessarie per connettersi alla risorsa e usarla dalle applicazioni sviluppate. In particolare:
    - Un *endpoint* HTTP a cui le applicazioni client possono inviare richieste.
    - Due *chiavi* che possono essere usate per l'autenticazione (le applicazioni client possono usare entrambe le chiavi per l'autenticazione).
    - La *località* in cui è ospitata la risorsa. Questa informazione è necessaria per le richieste ad alcune API, ma non a tutte.

## <a name="use-a-rest-interface"></a>Usare un'interfaccia REST

Le API di Servizi cognitivi sono basate su REST, quindi è possibile usarle inviando richieste JSON su HTTP. In questo esempio si esplorerà un'applicazione console che usa l'API REST **Analisi del testo** per eseguire il rilevamento della lingua, ma il principio di base è lo stesso per tutte le API supportate dalla risorsa di Servizi cognitivi.

> **Nota**: in questo esercizio è possibile scegliere se usare l'API REST da **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **01-getting-started** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Visualizzare il contenuto della cartella **rest-client** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.
4. Si noti che la cartella **rest-client** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: rest-client.py

    Aprire il file di codice ed esaminare il codice in esso contenuto, notando i dettagli seguenti:
    - Vengono importati vari spazi dei nomi per abilitare la comunicazione HTTP
    - Il codice nella funzione **Main** recupera l'endpoint e la chiave per la risorsa di Servizi cognitivi, che verranno usati per inviare richieste REST al servizio Analisi del testo.
    - Il programma accetta l'input dell'utente e usa la funzione **GetLanguage** per chiamare l'API REST Analisi del testo di rilevamento della lingua per l'endpoint di Servizi cognitivi, per rilevare la lingua del testo immesso.
    - La richiesta inviata all'API è costituita da un oggetto JSON contenente i dati di input, in questo caso una raccolta di oggetti **documento**, ognuno dei quali con un **ID** e un **testo**.
    - La chiave per il servizio è inclusa nell'intestazione della richiesta per autenticare l'applicazione client.
    - La risposta del servizio è un oggetto JSON che l'applicazione client può analizzare.
5. Fare clic con il pulsante destro del mouse sulla cartella **rest-client** e aprire un terminale integrato. Immettere quindi il comando specifico del linguaggio seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python rest-client.py
    ```

6. Quando richiesto, immettere un testo ed esaminare la lingua rilevata dal servizio, che viene restituita nella risposta JSON. Ad esempio, provare a immettere "Hello", "Bonjour" e "Hola".
7. Al termine del test dell'applicazione, immettere "quit" per arrestare il programma.

## <a name="use-an-sdk"></a>Usare un SDK

È possibile scrivere codice che utilizza direttamente le API REST di Servizi cognitivi, ma sono disponibili SDK (Software Development Kit) per molti linguaggi di programmazione diffusi, tra cui Microsoft C#, Python e Node.js. L'uso di un SDK può semplificare notevolmente lo sviluppo di applicazioni che utilizzano servizi cognitivi.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **01-getting-started** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **sdk-client** e aprire un terminale integrato. Installare quindi il pacchetto Text Analytics SDK eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.0.0
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.0.0
    ```

3. Visualizzare il contenuto della cartella **sdk-client** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.
    
4. Si noti che la cartella **sdk-client** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: sdk-client.py

    Aprire il file di codice ed esaminare il codice in esso contenuto, notando i dettagli seguenti:
    - Lo spazio dei nomi per l'SDK installato viene importato
    - Il codice nella funzione **Main** recupera l'endpoint e la chiave per la risorsa di Servizi cognitivi, che verranno usati con l'SDK per creare un client per il servizio Analisi del testo.
    - La funzione **GetLanguage** usa l'SDK per creare un client per il servizio e quindi usa il client per rilevare la lingua del testo immesso.
5. Tornare nel terminale integrato per la cartella **sdk-detector**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python sdk-client.py
    ```

6. Quando richiesto, immettere del testo ed esaminare la lingua rilevata dal servizio. Ad esempio, provare a immettere "Goodbye", "Au revoir" e "Hasta la vista".
7. Al termine del test dell'applicazione, immettere "quit" per arrestare il programma.

> **Nota**: alcune lingue che richiedono set di caratteri Unicode potrebbero non essere riconosciute in questa semplice applicazione console.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni su Servizi cognitivi di Azure, vedere la [documentazione di Servizi cognitivi](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services).

---
lab:
  title: Analizzare il testo
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: eb4e413e64c322f182cd5eadaff56bb880636e2f
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625994"
---
# <a name="analyze-text"></a>Analizzare il testo

L'**API Analisi del testo** è un servizio cognitivo che supporta l'analisi del testo, tra cui il rilevamento lingua, l'analisi valutazione, l'estrazione di frasi chiave e il riconoscimento di entità.

Si supponga, ad esempio, che un'agenzia di viaggi voglia elaborare recensioni di alberghi inviate al sito Web della società. L'API Analisi del testo consente di determinare la lingua in cui è scritta ogni recensione, la valutazione (positiva, neutra o negativa) delle recensioni, le frasi chiave che potrebbero indicare gli argomenti principali discussi nella recensione e le entità denominate, ad esempio luoghi, luoghi di interesse o persone citate nelle recensioni.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

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
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

## <a name="prepare-to-use-the-text-analytics-sdk"></a>Operazioni preliminari all'uso di Text Analytics SDK

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa Text Analytics SDK per analizzare le recensioni degli alberghi.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **05-analyze-text** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **text-analysis** e aprire un terminale integrato. Installare quindi il pacchetto Text Analytics SDK eseguendo il comando appropriato per il linguaggio scelto:
    
    **C#**
    
    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.0.0
    ```
    
    **Python**
    
    ```
    pip install azure-ai-textanalytics==5.0.0
    ```
    
3. Visualizzare il contenuto della cartella **text-analysis** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi cognitivi. Salvare le modifiche.

4. Si noti che la cartella **text-analysis** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: text-analysis.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico per la lingua per importare gli spazi dei nomi necessari per usare Text Analytics SDK:

    **C#**
    
    ```C#
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

5. Nella funzione **Main** si noti che il codice per caricare l'endpoint e la chiave di Servizi cognitivi dal file di configurazione è già stato fornito. Individuare quindi il commento **Create client using endpoint and key** e aggiungere il codice seguente per creare un client per l'API Analisi del testo:

    **C#**

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

6. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

6. Osservare l'output perché il codice deve essere eseguito senza errori, visualizzando i contenuti di ogni file di testo di recensione nella cartella **reviews**. L'applicazione crea correttamente un client per l'API Analisi del testo, ma non la usa. Questo problema verrà risolto nella procedura successiva.

## <a name="detect-language"></a>Rilevare la lingua

Ora che è stato creato un client per l'API Analisi del testo, è possibile usarlo per rilevare la lingua in cui viene scritta ogni recensione.

1. Nella funzione **Main** per il programma, individuare il commento **Get language**. Sotto questo commento aggiungere quindi il codice necessario per rilevare la lingua in ogni recensione:

    **C#**
    
    ```C
    // Get language
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**
    
    ```Python
    # Get language
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **Nota**: *in questo esempio ogni recensione viene analizzata singolarmente e di conseguenza viene effettuata una chiamata separata al servizio per ogni file. Un approccio alternativo consiste nel creare una raccolta di documenti e passarli al servizio in un'unica chiamata. In entrambi gli approcci, la risposta restituita dal servizio è costituita da una raccolta di documenti. È per questo motivo che nel codice Python precedente viene specificato l'indice del primo (e unico) documento incluso nella risposta ([0]).*

6. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

7. Osservare l'output, notando che questa volta viene identificata la lingua per ogni recensione.

## <a name="evaluate-sentiment"></a>Determinare la valutazione

L'*analisi valutazione* è una tecnica usata in genere per classificare il testo come *positivo* o *negativo* (oppure possibile *neutro* o *misto*). Viene usato in genere per analizzare post di social media, recensioni di prodotti e altri elementi in cui la valutazione del testo può fornire utili informazioni dettagliate.

1. Nella funzione **Main** per il programma, individuare il commento **Get sentiment**. Sotto questo commento aggiungere quindi il codice necessario per rilevare la valutazione di ogni recensione:

    **C#**
    
    ```C
    // Get sentiment
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**
    
    ```Python
    # Get sentiment
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

3. Osservare l'output, notando che viene rilevata la valutazione delle recensioni.

## <a name="identify-key-phrases"></a>Identificare le frasi chiave

Può essere utile identificare le frasi chiave nel corpo di un testo per determinare i principali argomenti trattati.

1. Nella funzione **Main** per il programma, individuare il commento **Get key phrases**. Sotto questo commento aggiungere quindi il codice necessario per rilevare le frasi chiave in ogni recensione:

    **C#**

    ```C
    // Get key phrases
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```
    
    **Python**
    
    ```Python
    # Get key phrases
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Osservare l'output, notando che ogni documento contiene frasi chiave che forniscono alcune informazioni dettagliate sul contenuto della recensione.

## <a name="extract-entities"></a>Estrarre le entità

Spesso nei documenti o in altri corpi del testo vengono citati persone, luoghi, periodi di tempo o altre entità. L'API Analisi del testo è in grado di rilevare più categorie (e sottocategorie) dell'entità nel testo.

1. Nella funzione **Main** per il programma, individuare il commento **Get entities**. Sotto questo commento aggiungere quindi il codice necessario per identificare le entità citate in ogni recensione:

    **C#**
    
    ```C
    // Get entities
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get entities
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Osservare l'output, notando le entità che sono state rilevate nel testo.

## <a name="extract-linked-entities"></a>Estrarre entità collegate

Oltre alle entità classificate, l'API Analisi del testo è in grado di rilevare le entità per cui sono presenti collegamenti noti alle origini dati, ad esempio Wikipedia.

1. Nella funzione **Main** per il programma, individuare il commento **Get linked entities**. Sotto questo commento aggiungere quindi il codice necessario per identificare le entità collegate citate in ogni recensione:

    **C#**
    
    ```C
    // Get linked entities
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**
    
    ```Python
    # Get linked entities
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **text-analysis**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. Osservare l'output, notando le entità collegate che sono state identificate.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'uso del servizio **Analisi del testo**, vedere la [documentazione di Analisi del testo](https://docs.microsoft.com/azure/cognitive-services/text-analytics/).

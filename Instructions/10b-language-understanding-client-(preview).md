---
lab:
  title: Creare un'applicazione client di comprensione del linguaggio di conversazione (anteprima)
ms.openlocfilehash: 486a6716a189156739e9107824e561f7bb8fa95c
ms.sourcegitcommit: 2c92ea1491cac72a4765755cf2bc6d8b5f657a46
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/09/2022
ms.locfileid: "141580026"
---
# <a name="create-a-language-service-client-application"></a>Creare un'applicazione client basata sul servizio di linguaggio

La funzionalità di comprensione del linguaggio di conversazione di Servizio cognitivo di Azure per la lingua consente di definire un modello linguistico per conversazioni che le app possono usare per interpretare l'input in linguaggio naturale degli utenti, stimare la *finalità* degli utenti, ossia cosa vogliono ottenere, e identificare le *entità* a cui applicare la finalità. È possibile creare applicazioni client che utilizzano modelli di comprensione del linguaggio di conversazione direttamente, tramite interfacce REST, oppure usando SDK (Software Development Kit) specifici del linguaggio.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se è già stato clonato il repository di codice **AI-102-AIEngineer** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.

2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).

3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-language-service-resources"></a>Creare risorse del servizio di linguaggio

Se nella sottoscrizione di Azure è già disponibile una risorsa del servizio di linguaggio, è possibile usarla in questo esercizio. In caso contrario, seguire queste istruzioni per crearne una.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *servizio di linguaggio* e creare una risorsa del **Servizio Lingua** con le impostazioni seguenti:

    - **Funzionalità predefinite**: tutte
    - **Funzionalità personalizzate**: nessuna
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, è possibile che non si sia autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *westus2*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: *S*
    - **Note legali**: *selezionare la casella di controllo per confermare*
    - **Avviso del Responsabile IA**: *selezionare la casella di controllo per confermare*

3. Attendere che le risorse vengano create. È possibile visualizzare la risorsa accedendo al gruppo di risorse in cui è stata creata.

## <a name="import-train-and-publish-a-conversational-language-understanding-model"></a>Importare un modello di comprensione del linguaggio di conversazione, eseguirne il training e pubblicarlo

Se si ha già un progetto **Clock** creato in un esercizio o lab precedente, è possibile usarlo in questo esercizio. In caso contrario, seguire queste istruzioni per crearla.

1. In una nuova scheda del browser aprire il portale di Language Studio - Preview all'indirizzo `https://language.cognitive.azure.com`.

2. Accedere usando l'account Microsoft associato alla sottoscrizione di Azure. Se è la prima volta che si accede al portale del servizio di linguaggio, potrebbe essere necessario concedere all'app alcune autorizzazioni in modo che possa accedere ai dettagli dell'account. Completare quindi i passaggi di *benvenuto* selezionando la sottoscrizione di Azure e la risorsa di creazione appena creata.

3. Aprire la pagina **Conversational Language Understanding**.

3. Accanto a **&#65291;Create new project** visualizzare l'elenco a discesa e selezionare **Import**. Fare clic su **Choose File** e quindi passare alla sottocartella **10b-clu-client-(preview)** nella cartella del progetto contenente i file lab per questo esercizio. Selezionare **Clock.json**, fare clic su **Open** e quindi su **Done**.

4. Se compare un pannello con consigli per creare un'app del servizio di linguaggio efficace, chiuderla.

5. A sinistra del portale di Language Studio selezionare **Train model** per eseguire il training dell'app. Fare clic su **Start a training job**, assegnare al modello il nome **Clock** e verificare che la valutazione con training sia abilitata. Il training può richiede alcuni minuti.

    > **Nota**: poiché il nome del modello **Clock** è hardcoded nel codice clock-client (usato più avanti nel lab), scriverlo con l'iniziale maiuscola ed esattamente come è descritto. 

6. A sinistra del portale di Language Studio selezionare **Deploy model** e usare **Add deployment** per creare la distribuzione per il modello Clock denominato **production**.

    > **Nota**: poiché il nome della distribuzione **production** è hardcoded nel codice clock-client (usato più avanti nel lab), scriverlo con l'iniziale maiuscola ed esattamente come è descritto. 

7. Al termine della distribuzione, selezionare la distribuzione **production** e quindi fare clic su **Get prediction URL**. Le applicazioni client necessitano dell'URL dell'endpoint per usare il modello distribuito.

8. A sinistra del portale di Language Studio selezionare **Project Settings** e prendere nota del valore di **Primary key**. Le applicazioni client necessitano della chiave primaria per usare il modello distribuito.

9. Le applicazioni client necessitano delle informazioni dall'URL di stima e dalla chiave del servizio di linguaggio per connettersi al modello distribuito ed essere autenticate.

## <a name="prepare-to-use-the-language-service-sdk"></a>Prepararsi a usare Language service SDK

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa il modello Clock (modello di comprensione del linguaggio di conversazione pubblicato) per stimare le finalità dall'input dell'utente e rispondere in modo appropriato.

> **Nota**: è possibile scegliere di usare l'SDK per **.NET** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **10b-clu-client-(preview)** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.

2. Fare clic con il pulsante destro del mouse sulla cartella **clock-client** e quindi scegliere **Apri nel terminale integrato**. Installare quindi il pacchetto Conversational Language Service SDK eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-language-conversations --pre
    python -m pip install python-dotenv
    python -m pip install python-dateutil
    ```

3. Visualizzare il contenuto della cartella **clock-client** e notare che include un file per le impostazioni di configurazione:

    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**URL endpoint** e la **chiave primaria** della risorsa Linguaggio. È possibile trovare i valori obbligatori nel portale di Azure o in Language Studio come indicato di seguito:

    - Portale di Azure: Aprire la risorsa Linguaggio. In **Gestione risorse** selezionare **Chiavi ed endpoint**. Copiare i valori **CHIAVE 1** ed **Endpoint** nel file delle impostazioni di configurazione.
    - Language Studio: Aprire il progetto **Clock**. L'endpoint di servizio di linguaggio si trova nella pagina **Deploy model** in **Get prediction URL**, mentre il valore di **Primary key** è disponibile nella pagina **Project settings**. La parte dell'URL di stima relativa all'endpoint di servizio di linguaggio termina con **.cognitiveservices.azure.com/** . Ad esempio: `https://ai102-langserv.cognitiveservices.azure.com/`.

4. Si noti che la cartella **clock-client** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: clock-client.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Language service SDK:

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    from azure.ai.language.conversations.models import ConversationAnalysisOptions
    ```

## <a name="get-a-prediction-from-the-conversational-language-model"></a>Ottenere una stima dal modello linguistico per conversazioni

A questo punto è possibile implementare il codice che usa l'SDK per ottenere una stima dal modello linguistico per conversazioni.

1. Nella funzione **Main** si noti che il codice per caricare l'endpoint di previsione e la chiave dal file di configurazione è già stato fornito. Trovare quindi il commento **Create a client for the Language service model** e aggiungere il codice seguente per creare un client di previsione per l'app Language Service:

    **C#**

    ```C#
    // Create a client for the Language service model
    Uri lsEndpoint = new Uri(predictionEndpoint);
    AzureKeyCredential lsCredential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient lsClient = new ConversationAnalysisClient(lsEndpoint, lsCredential);
    ```

    **Python**

    ```Python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

2. Si noti che il codice nella funzione **Main** richiede input da parte dell'utente finché l'utente non immette "quit". All'interno di questo ciclo trovare il commento **Call the Language service model to get intent and entities** e aggiungere il codice seguente:

    **C#**

    ```C#
    // Call the Language service model to get intent and entities
    var lsProject = "Clock";
    var lsSlot = "production";

    ConversationsProject conversationsProject = new ConversationsProject(lsProject, lsSlot);
    Response<AnalyzeConversationResult> response = await lsClient.AnalyzeConversationAsync(userText, conversationsProject);

    ConversationPrediction conversationPrediction = response.Value.Prediction as ConversationPrediction;

    Console.WriteLine(JsonConvert.SerializeObject(conversationPrediction, Formatting.Indented));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].Confidence > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }

    var entities = conversationPrediction.Entities;
    ```

    **Python**

    ```Python
    # Call the Language service model to get intent and entities
    convInput = ConversationAnalysisOptions(
        query = userText
        )
    
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        result = client.analyze_conversations(
            convInput, 
            project_name=cls_project, 
            deployment_name=deployment_slot
            )

    # list the prediction results
    top_intent = result.prediction.top_intent
    entities = result.prediction.entities

    print("view top intent:")
    print("\ttop intent: {}".format(result.prediction.top_intent))
    print("\tcategory: {}".format(result.prediction.intents[0].category))
    print("\tconfidence score: {}\n".format(result.prediction.intents[0].confidence_score))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity.category))
        print("\ttext: {}".format(entity.text))
        print("\tconfidence score: {}".format(entity.confidence_score))

    print("query: {}".format(result.query))
    ```

    La chiamata al modello di servizio di linguaggio restituisce una previsione o un risultato, che include la finalità principale (più probabile) e tutte le entità rilevate nell'espressione di input. L'applicazione client deve ora usare la previsione per determinare ed eseguire l'azione appropriata.

3. Trovare il commento **Apply the appropriate action** e aggiungere il codice seguente, che verifica la presenza di finalità supportate dall'applicazione (**GetTime**, **GetDate** e **GetDay**) e determina se sono state rilevate entità pertinenti, prima di chiamare una funzione esistente per produrre una risposta appropriata.

    **C#**

    ```C#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a location entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Location")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        location = entity.Text;
                    }
                    
                }

            }

            // Get the time for the specified location
            var getTimeTask = Task.Run(() => GetTime(location));
            string timeResponse = await getTimeTask;
            Console.WriteLine(timeResponse);
            break;

        case "GetDay":
            var date = DateTime.Today.ToShortDateString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Date entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Date")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        date = entity.Text;
                    }
                }
            }
            // Get the day for the specified date
            var getDayTask = Task.Run(() => GetDay(date));
            string dayResponse = await getDayTask;
            Console.WriteLine(dayResponse);
            break;

        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Weekday entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Weekday")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        day = entity.Text;
                    }
                }
                
            }
            // Get the date for the specified day
            var getDateTask = Task.Run(() => GetDate(day));
            string dateResponse = await getDateTask;
            Console.WriteLine(dateResponse);
            break;

        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**

    ```Python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity.category:
                    # ML entities are strings, get the first one
                    location = entity.text
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity.category:
                    # Regex entities are strings, get the first one
                    date_string = entity.text
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity.category:
                # List entities are lists
                    day = entity.text
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

4. Salvare le modifiche e tornare al terminale integrato per la cartella **clock-client**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python clock-client.py
    ```

5. Quando richiesto, immettere espressioni per testare l'applicazione. Ad esempio, provare:

    *Ciao*
    
    *What time is it?*

    *What's the time in London?*

    *What's the date?*

    *What date is Sunday?*

    *What day is it?*

    *What day is 01/01/2025?*

> **Nota**: la logica nell'applicazione è volutamente semplice e presenta una serie di limitazioni. Ad esempio, per il recupero dell'ora è supportato solo un set limitato di città e l'ora legale viene ignorata. Lo scopo è vedere un esempio di un modello tipico di uso del servizio di linguaggio in cui l'applicazione deve:
>
>   1. Connettersi a un endpoint di previsione.
>   2. Inviare un'espressione per ottenere una previsione.
>   3. Implementare logica per rispondere in modo appropriato alla finalità prevista e alle entità.

6. Al termine del test, immettere *quit*.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla creazione di un client del servizio di linguaggio, vedere la [documentazione per sviluppatori](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)

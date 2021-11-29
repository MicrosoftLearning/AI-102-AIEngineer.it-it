---
lab:
  title: Usare i servizi Voce e Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 31b88612bce38780f828859f8e2b166dbe8df34d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625982"
---
# <a name="use-the-speech-and-language-understanding-services"></a>Usare i servizi Voce e Language Understanding

È possibile integrare il servizio Voce con il servizio Language Understanding per creare applicazioni in grado di determinare in modo intelligente le finalità dell'utente dall'input vocale.

> **Nota**: questo esercizio funziona al meglio se si ha un microfono. Alcuni ambienti virtuali ospitati potrebbero essere in grado di acquisire audio dal microfono locale, ma se questo approccio non funziona o non è disponibile alcun microfono, è possibile usare un file audio fornito per l'input vocale. Seguire con attenzione le istruzioni, poiché sarà necessario scegliere opzioni diverse in base alla scelta di usare un microfono oppure il file audio.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se è già stato clonato il repository di codice **AI-102-AIEngineer** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-language-understanding-resources"></a>Creare risorse di Language Understanding

Se si hanno già risorse di creazione e previsione di Language Understanding nella sottoscrizione di Azure, è possibile usarle in questo esercizio. In caso contrario, seguire queste istruzioni per crearle.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&amp;#65291;Crea una risorsa**, cercare *language understanding* e creare una risorsa di **Language Understanding** con le impostazioni seguenti:
    - **Opzione Crea**: entrambe
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Nome**: *immettere un nome univoco*
    - **Posizione di creazione**: *selezionare la posizione preferita*
    - **Piano tariffario di creazione**: F0
    - **Percorso di stima**: *scegliere la <u>stessa posizione</u> di creazione*
    - **Piano tariffario per le previsioni**: F0 (*se F0 non è disponibile, scegliere S0*)

3. Attendere che vengano create le risorse e tenere presente che è stato effettuato il provisioning di due risorse di Language Understanding, una per la creazione e un'altra per la previsione. È possibile visualizzarle entrambe passando al gruppo di risorse in cui sono state create.

## <a name="prepare-a-language-understanding-app"></a>Preparare un'app basata su Language Understanding

Se si ha già un'app **Orologio** creata in un esercizio precedente, aprirla nel portale di Language Understanding all'indirizzo `https://www.luis.ai`. In caso contrario, seguire queste istruzioni per crearla.

1. In una nuova scheda del browser aprire il portale di Language Understanding all'indirizzo `https://www.luis.ai`.
2. Accedere usando l'account Microsoft associato alla sottoscrizione di Azure. Se è la prima volta che accedi al portale di Language Understanding potresti dover concedere all'app alcuni permessi in modo che possa accedere ai dettagli del tuo account. Completare quindi i passaggi di *benvenuto* selezionando la sottoscrizione di Azure e la risorsa di creazione appena creata.
3. Aprire la pagina **Conversation Apps** (App di conversazione). Accanto a **&#65291;Nuova app** visualizzare l'elenco a discesa e selezionare **Importa come LU**.
Passare alla sottocartella **11-luis-speech** nella cartella del progetto contenente i file del lab per questo esercizio e selezionare **Clock.lu**. Specificare quindi un nome univoco per l'app orologio.
4. Se compare un riquadro con dei consigli per creare un'app di Language Understanding efficace, chiudila.

## <a name="train-and-publish-the-app-with-speech-priming"></a>Eseguire il training e pubblicare l'app con il *priming vocale*

1. Nella parte superiore del portale di Language Understanding selezionare **Esegui il training**  per eseguire il training dell'app, se non è già stato fatto.
2. Nella parte superiore del portale di Language Understanding selezionare **Pubblica**. Selezionare quindi lo **Slot di produzione** e modificare le impostazioni per abilitare il **Priming vocale**. Ne deriverà un miglioramento delle prestazioni di riconoscimento vocale.
3. Al termine della pubblicazione, nella parte superiore del portale di Language Understanding selezionare **Gestisci**.
4. Nella pagina **Impostazioni** annotare l'**ID app**. Le applicazioni client ne avranno bisogno per usare l'app.
5. Nella pagina **Risorse di Azure**, se in **Risorse di previsione** non è elencata alcuna risorsa, aggiungere la risorsa di previsione nella sottoscrizione di Azure.
6. Annotare la **Chiave primaria**, la **Chiave secondaria** e la **Località** (<u>non</u> l'endpoint) della risorsa di previsione. Le applicazioni client di Speech SDK hanno bisogno della posizione e di una delle chiavi per connettersi alla risorsa di previsione ed essere autenticate.

## <a name="configure-a-client-application-for-language-understanding"></a>Configurare un'applicazione client per Language Understanding

In questo esercizio si creerà un'applicazione client che accetta l'input vocale e usa l'app di Language Understanding per prevedere la finalità dell'utente.

> **Nota**: in questo esercizio si può scegliere di usare l'SDK per **C#** o per **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **11-luis-speech** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Visualizzare il contenuto della cartella **speaking-clock-client** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori di configurazione al suo interno per includere l'**ID** dell'app di Language Understanding, la sua **località** (<u>non</u> l'endpoint completo, ad esempio *eastus*) e una delle **chiavi** della risorsa di previsione corrispondente (dalla pagina **Gestisci** per l'app nel portale di Language Understanding).

## <a name="install-sdk-packages"></a>Installare i pacchetti SDK

Per usare Speech SDK con il servizio Language Understanding, è necessario installare il pacchetto Speech SDK per il linguaggio di programmazione scelto.

1. In Visual Studio fare clic con il pulsante destro del mouse sulla cartella **speaking-clock-client** e aprire un terminale integrato. Installare quindi il pacchetto Language Understanding SDK eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. Inoltre, se il sistema <u>non</u> dispone di un microfono funzionante, sarà necessario usare un file audio per fornire l'input vocale per l'applicazione. In questo caso, usare i comandi seguenti per installare un pacchetto aggiuntivo in modo che il programma possa riprodurre il file audio (se si intende usare un microfono, è possibile ignorare questo passaggio):

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. Si noti che la cartella **speaking-clock-client** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: speaking-clock-client.py

4. Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Speech SDK:

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. Inoltre, se il sistema <u>non</u> dispone di un microfono funzionante, sotto le importazioni dello spazio dei nomi esistenti aggiungere il codice che segue per importare la libreria da usare per la riproduzione di un file audio:

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## <a name="create-an-intentrecognizer"></a>Creare un oggetto *IntentRecognizer*

La classe **IntentRecognizer** fornisce un oggetto client che è possibile usare per ottenere previsioni di Language Understanding dall'input vocale.

1. Nella funzione **Main** si noti che il codice per caricare l'ID app, l'area di previsione e la chiave dal file di configurazione è già stato fornito. Trovare quindi il commento **Configure speech service and get intent recognizer** e aggiungere il codice seguente a seconda che si preveda di usare un microfono o un file audio per l'input vocale:

    ### <a name="if-you-have-a-working-microphone"></a>**Se è disponibile un microfono funzionante:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### <a name="if-you-need-to-use-an-audio-file"></a>**Se è necessario usare un file audio:**

    **C#**

    ```C#
    // Configure speech service and get intent recognizer
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech service and get intent recognizer
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## <a name="get-a-predicted-intent-from-spoken-input"></a>Ottenere una finalità prevista dall'input vocale

A questo punto si è pronti per implementare codice che usa Speech SDK per ottenere una finalità prevista dall'input vocale.

1. Nella funzione **Main**, immediatamente sotto il codice appena aggiunto individuare il commento **Get the model from the AppID and add the intents we want to use** e aggiungere il codice seguente per ottenere il modello Language Understanding (in base all'ID app) e specificare le finalità che lo strumento di riconoscimento deve identificare.

    **C#**

    ```C#
    // Get the model from the AppID and add the intents we want to use
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *Si noti che è possibile specificare un ID basato su stringa per ogni finalità*

    **Python**

    ```Python
    # Get the model from the AppID and add the intents we want to use
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. Sotto il commento **Process speech input** aggiungere il codice seguente, che usa lo strumento di riconoscimento per chiamare in modo asincrono il servizio Language Understanding con input vocale e recuperare la risposta. Se la risposta include una finalità prevista, vengono visualizzate la query pronunciata, la finalità prevista e la risposta JSON completa. In caso contrario, il codice gestisce la risposta in base al motivo restituito.

**C#**

```C
// Process speech input
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // Intent was identified
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // Get the first entity (if any)

    // Apply the appropriate action
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // Speech was recognized, but no intent was identified.
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // Speech wasn't recognized
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // Something went wrong
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# Process speech input
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # Get the first entity (if any)
    
    # Apply the appropriate action
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # Speech was recognized, but no intent was identified.
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # Speech wasn't recognized
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # Something went wrong
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

Il codice aggiunto finora identifica la *finalità*, ma alcune finalità possono fare riferimento a *entità*, pertanto è necessario aggiungere codice per estrarre le informazioni sull'entità dal codice JSON restituito dal servizio.

3. Nel codice appena aggiunto individuare il commento **Get the first entity (if any)** e aggiungere il codice seguente sotto di esso:

**C#**

```C
// Get the first entity (if any)
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# Get the first entity (if any)
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
Il codice ora usa l'app di Language Understanding per prevedere una finalità e tutte le entità rilevate nell'espressione di input. L'applicazione client deve ora usare la previsione per determinare ed eseguire l'azione appropriata.

4. Sotto il codice appena aggiunto trovare il commento **Apply the appropriate action** e aggiungere il codice seguente, che verifica la presenza di finalità supportate dall'applicazione (**GetTime**, **GetDate** e **GetDay**) e determina se sono state rilevate entità pertinenti, prima di chiamare una funzione esistente per produrre una risposta appropriata.

**C#**

```C#
// Apply the appropriate action
switch (intent)
{
    case "time":
        var location = "local";
        // Check for entities
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // Get the time for the specified location
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // Check for entities
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // Get the day for the specified date
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // Check for entities
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // Some other intent (for example, "None") was predicted
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# Apply the appropriate action
if intent == 'GetTime':
    location = 'local'
    # Check for entities
    if entity_type == 'Location':
        location = entity_value
    # Get the time for the specified location
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if entity_type == 'Date':
        date_string = entity_value
    # Get the day for the specified date
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # Check for entities
    if entity_type == 'Weekday':
        # List entities are lists
        day = entity_value
    # Get the date for the specified day
    print(GetDate(day))

else:
    # Some other intent (for example, "None") was predicted
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## <a name="run-the-client-application"></a>Eseguire l'applicazione client

1. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock-client**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. Se si usa un microfono, pronunciare espressioni ad alta voce per testare l'applicazione. Ad esempio, provare le seguenti, eseguendo di nuovo il programma ogni volta:

    *What's the time?*
    
    *What time is it?*

    *What day is it?*

    *What is the time in London?*

    *What's the date?*

    *What date is Sunday?*

> **Nota**: la logica nell'applicazione è volutamente semplice e presenta una serie di limitazioni, ma il suo scopo è semplicemente testare la capacità del modello Language Understanding di prevedere finalità dall'input vocale usando Speech SDK. Si potrebbero riscontrare problemi con il riconoscimento della finalità **GetDay** con una specifica entità di data, per la difficoltà di verbalizzare una data in formato *MM/GG/AAAA*.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'integrazione dei servizi Voce e Language Understanding, vedere la [documentazione di Voce](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition).

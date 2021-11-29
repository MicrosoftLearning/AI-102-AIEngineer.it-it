---
lab:
  title: Creare un'app client basata su Language Understanding
  module: Module 5 - Creating Language Understanding Solutions
ms.openlocfilehash: 36f75e47910707c959495140be43c4196649d471
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625826"
---
# <a name="create-a-language-understanding-client-application"></a>Creare un'app client basata su Language Understanding

Il servizio Language Understanding consente di definire un'app che incapsula un modello linguistico che le applicazioni possono usare per interpretare l'input in linguaggio naturale degli utenti, prevedere la *finalità* degli utenti, ossia cosa vogliono ottenere, e identificare le *entità* a cui applicare la finalità. È possibile creare applicazioni client che utilizzano app di Language Understanding direttamente, tramite interfacce REST, oppure utilizzando SDK (Software Development Kit) specifici del linguaggio.

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

## <a name="import-train-and-publish-a-language-understanding-app"></a>Importare, sottoporre a training e pubblicare un'app di Language Understanding

Se si ha già un'app **Orologio** creata in un esercizio precedente, è possibile usarla in questo esercizio. In caso contrario, seguire queste istruzioni per crearla.

1. In una nuova scheda del browser aprire il portale di Language Understanding all'indirizzo `https://www.luis.ai`.
2. Accedere usando l'account Microsoft associato alla sottoscrizione di Azure. Se è la prima volta che accedi al portale di Language Understanding potresti dover concedere all'app alcuni permessi in modo che possa accedere ai dettagli del tuo account. Completare quindi i passaggi di *benvenuto* selezionando la sottoscrizione di Azure e la risorsa di creazione appena creata.
3. Aprire la pagina **Conversation Apps** (App di conversazione). Accanto a **&#65291;Nuova app** visualizzare l'elenco a discesa e selezionare **Importa come LU**.
Passare alla sottocartella **10-luis-client** nella cartella del progetto contenente i file del lab per questo esercizio e selezionare **Clock.lu**. Specificare quindi un nome univoco per l'app orologio.
4. Se compare un riquadro con dei consigli per creare un'app di Language Understanding efficace, chiudila.
5. Nella parte superiore del portale di Language Understanding selezionare **Esegui il training**  per eseguire il training dell'app.
6. In alto a destra nel portale di Language Understanding selezionare **Pubblica** e pubblicare l'app nello **Slot di produzione**.
7. Al termine della pubblicazione, nella parte superiore del portale di Language Understanding selezionare **Gestisci**.
8. Nella pagina **Impostazioni** annotare l'**ID app**. Le applicazioni client ne avranno bisogno per usare l'app.
9. Nella pagina **Risorse di Azure**, se in **Risorse di previsione** non è elencata alcuna risorsa, aggiungere la risorsa di previsione nella sottoscrizione di Azure.
10. Annotare la **Chiave primaria**, la **Chiave secondaria** e l'**URL dell'endpoint** per la risorsa di previsione. Le applicazioni client hanno bisogno dell'endpoint e di una delle chiavi per connettersi alla risorsa di previsione ed essere autenticate.

## <a name="prepare-to-use-the-language-understanding-sdk"></a>Prepararsi a usare Language Understanding SDK

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa l'app Orologio di Language Understanding per stimare le finalità dall'input dell'utente e rispondere in modo appropriato.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **10-luis-client** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **clock-client** e aprire un terminale integrato. Installare quindi il pacchetto Language Understanding SDK eseguendo il comando appropriato per il linguaggio scelto:

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

*Oltre al pacchetto **Runtime** (previsione), è disponibile un pacchetto **Authoring** che è possibile usare per scrivere codice per creare e gestire modelli Language Understanding.*

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
*Il pacchetto Python SDK include classi sia per la **previsione** che per la **creazione**.*

3. Visualizzare il contenuto della cartella **clock-client** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori di configurazione al suo interno per includere l'**ID** dell'app di Language Understanding, l'**URL dell'endpoint** e una delle **chiavi** della risorsa di previsione corrispondente (dalla pagina **Gestisci** per l'app nel portale di Language Understanding).

4. Si noti che la cartella **clock-client** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: clock-client.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. Quindi, sotto questo commento aggiungere il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare l'SDK di previsione di Language Understanding:

**C#**

```C#
// Import namespaces
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# Import namespaces
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## <a name="get-a-prediction-from-the-language-understanding-app"></a>Ottenere una previsione dall'app di Language Understanding

A questo punto è possibile implementare codice che usa l'SDK per ottenere una previsione dall'app di Language Understanding.

1. Nella funzione **Main** si noti che il codice per caricare l'ID app, l'endpoint di previsione e la chiave dal file di configurazione è già stato fornito. Trovare quindi il commento **Create a client for the LU app** e aggiungere il codice seguente per creare un client di previsione per l'app di Language Understanding:

**C#**

```C#
// Create a client for the LU app
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# Create a client for the LU app
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. Si noti che il codice nella funzione **Main** richiede input da parte dell'utente finché l'utente non immette "quit". All'interno di questo ciclo trovare il commento **Call the LU app to get intent and entities** e aggiungere il codice seguente:

**C#**

```C#
// Call the LU app to get intent and entities
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# Call the LU app to get intent and entities
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

La chiamata all'app di Language Understanding restituisce una previsione, che include la finalità principale (più probabile) e tutte le entità rilevate nell'espressione di input. L'applicazione client deve ora usare la previsione per determinare ed eseguire l'azione appropriata.

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
            if (entities.ContainsKey("Location"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML entities are strings, get the first one
                location = entityJson[0].ToString();
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
            if (entities.ContainsKey("Date"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // Regex entities are strings, get the first one
                date = entityJson[0].ToString();
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
            if (entities.ContainsKey("Weekday"))
            {
                //Get the JSON for the entity
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // List entities are lists
                day = entityJson[0][0].ToString();
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
        if 'Location' in entities:
            # ML entities are strings, get the first one
            location = entities['Location'][0]
    # Get the time for the specified location
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # Check for entities
    if len(entities) > 0:
        # Check for a Date entity
        if 'Date' in entities:
            # Regex entities are strings, get the first one
            date_string = entities['Date'][0]
    # Get the day for the specified date
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # Check for entities
    if len(entities) > 0:
        # Check for a Weekday entity
        if 'Weekday' in entities:
            # List entities are lists
            day = entities['Weekday'][0][0]
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

> **Nota**: la logica nell'applicazione è volutamente semplice e presenta una serie di limitazioni. Ad esempio, per il recupero dell'ora è supportato solo un set limitato di città e l'ora legale viene ignorata. Lo scopo è vedere un esempio di un modello tipico di utilizzo del servizio Language Understanding in cui l'applicazione deve:
>
>   1. Connettersi a un endpoint di previsione.
>   2. Inviare un'espressione per ottenere una previsione.
>   3. Implementare logica per rispondere in modo appropriato alla finalità prevista e alle entità.

6. Al termine del test, immettere *quit*.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla creazione di un client di Language Understanding client, vedere la [documentazione per sviluppatori](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)

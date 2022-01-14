---
lab:
  title: Tradurre il testo
  module: Module 3 - Getting Started with Natural Language Processing
ms.openlocfilehash: 4ca4394ce5d9456abeabbdded1ee219f271f284e
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625827"
---
# <a name="translate-text"></a>Tradurre il testo

Il servizio **Traduttore** è un servizio cognitivo che consente di tradurre testo tra lingue diverse.

Si supponga ad esempio che un'agenzia di viaggi voglia esaminare recensioni di hotel inviate al sito Web della società, specificando l'inglese come lingua standard usata per l'analisi. Grazie al servizio Traduttore, l'agenzia può determinare la lingua in cui è stata scritta ogni recensione e, se non è già in inglese, tradurla in inglese da qualsiasi lingua di origine usata per scriverla.

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
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, è possibile che non si sia autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
5. Al termine della distribuzione della risorsa, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva sarà necessario usare una delle chiavi e la posizione in cui è stato effettuato il provisioning del servizio, disponibili in questa pagina.

## <a name="prepare-to-use-the-translator-service"></a>Preparare l'uso del servizio Traduttore

In questo esercizio verrà completata un'applicazione client parzialmente implementata che usa l'API REST Traduttore per tradurre recensioni di hotel.

> **Nota**: è possibile scegliere di usare l'API per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **06-translate-text** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Visualizzare i contenuti della cartella **text-translation** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori di configurazione contenuti nel file in modo da includere una **chiave** di autenticazione per la risorsa di Servizi cognitivi e la **posizione** in cui è distribuita (<u>non</u> l'endpoint). È consigliabile copiare entrambi i valori dalla pagina **keys and Endpoint** (Chiavi ed endpoint) per la risorsa di Servizi cognitivi. Salvare le modifiche.
3. Si noti che la cartella **text-translation** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: text-translation.py

    Aprire il file di codice ed esaminare il codice contenuto nel file.

4. Nella funzione **Main** si noti che il codice per caricare la chiave e l'area di Servizi cognitivi dal file di configurazione è già stato fornito. L'endpoint per il servizio di traduzione è specificato anche nel codice.
5. Fare clic con il pulsante destro del mouse sulla cartella **text-translation**, aprire un terminale integrato e immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

6. Osservare l'output perché il codice deve essere eseguito senza errori, visualizzando i contenuti di ogni file di testo di recensione nella cartella **reviews**. L'applicazione non usa attualmente il servizio Traduttore. Questo problema verrà risolto nella procedura successiva.

## <a name="detect-language"></a>Rilevare la lingua

Il servizio Traduttore può rilevare automaticamente la lingua di origine del testo da tradurre, ma consente anche di rilevare esplicitamente la lingua in cui è scritto il testo.

1. Nel file di codice trovare la funzione **GetLanguage**, che restituisce attualmente "en" per tutti i valori di testo.
2. Nella funzione **GetLanguage**, sotto il commento **Use the Translator detect function**, aggiungere il codice seguente per usare l'API REST Traduttore per rilevare la lingua del testo specificato, prestando attenzione a non sostituire il codice alla fine della funzione che restituisce la lingua:

**C#**

```C#
// Use the Translator detect function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/detect?api-version=3.0";
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get language
        JArray jsonResponse = JArray.Parse(responseContent);
        language = (string)jsonResponse[0]["language"]; 
    }
}
```

**Python**

```Python
# Use the Translator detect function
path = '/detect'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0'
}

headers = {
'Ocp-Apim-Subscription-Key': cog_key,
'Ocp-Apim-Subscription-Region': cog_region,
'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get language
language = response[0]["language"]
```

3. Salvare le modifiche e tornare al terminale integrato per la cartella **text-translation**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. Osservare l'output, notando che questa volta viene identificata la lingua per ogni recensione.

## <a name="translate-text"></a>Tradurre il testo

Ora che l'applicazione può determinare la lingua in cui sono scritte le recensioni, è possibile usare il servizio Traduttore per tradurre in inglese eventuali recensioni in lingue diverse dall'inglese.

1. Nel file di codice trovare la funzione **Translate**, che restituisce attualmente una stringa vuota per tutti i valori di testo.
2. Nella funzione **Translate**, sotto il commento **Use the Translator translate function**, aggiungere il codice seguente per usare l'API REST Traduttore per tradurre il testo specificato dalla lingua di origine all'inglese, prestando attenzione a non sostituire il codice alla fine della funzione che restituisce la traduzione:

**C#**

```C#
// Use the Translator translate function
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // Build the request
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // Send the request and get response
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // Read response as a string
        string responseContent = await response.Content.ReadAsStringAsync();

        // Parse JSON array and get translation
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```Python
# Use the Translator translate function
path = '/translate'
url = translator_endpoint + path

# Build the request
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# Send the request and get response
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# Parse JSON array and get translation
translation = response[0]["translations"][0]["text"]
```

3. Salvare le modifiche e tornare al terminale integrato per la cartella **text-translation**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. Osservare l'output, notando che le revisioni non in inglese vengono tradotte in inglese.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'uso del servizio **Traduttore**, vedere la [documentazione di Traduttore](https://docs.microsoft.com/azure/cognitive-services/translator/).

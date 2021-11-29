---
lab:
  title: Riconoscimento e sintesi vocale
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 88f5d500dc5adad83bdd476a278329792e1b2e45
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625820"
---
# <a name="recognize-and-synthesize-speech"></a>Riconoscimento e sintesi vocale

Il servizio **Voce** è un servizio cognitivo di Azure che offre funzionalità correlate al riconoscimento vocale, tra cui:

- Un'API di *conversione della voce in testo scritto* che consente di implementare il riconoscimento vocale (convertendo parole vocali udibili in testo).
- Un'API *sintesi vocale* che consente di implementare la sintesi vocale (convertendo il testo in voce udibile).

In questo esercizio si useranno entrambe queste API per implementare un'applicazione di orologio vocale.

**Nota**: per questo esercizio è necessario usare un computer con altoparlanti/cuffia. Per un'esperienza ottimale è necessario anche un microfono. Alcuni ambienti virtuali ospitati potrebbero essere in grado di acquisire audio dal microfono locale, ma se questo approccio non funziona o non è disponibile alcun microfono, è possibile usare un file audio fornito per l'input vocale. Seguire con attenzione le istruzioni, poiché sarà necessario scegliere opzioni diverse in base alla scelta di usare un microfono oppure il file audio.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

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
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva sarà necessario usare una delle chiavi e la posizione in cui è stato effettuato il provisioning del servizio, disponibili in questa pagina.

## <a name="prepare-to-use-the-speech-service"></a>Preparare l'uso del servizio Voce

In questo esercizio verrà completata un'applicazione client parzialmente implementata che usa Speech SDK per riconoscere e sintetizzare il parlato.

**Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Explorer** di Visual Studio Code passare alla cartella **07-speech** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **speaking-clock** e aprire un terminale integrato. Installare quindi il pacchetto Speech SDK eseguendo il comando appropriato per la lingua scelta:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

3. Visualizzare il contenuto della cartella **speaking-clock** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori di configurazione inclusi nel file in modo che includano una **chiave** di autenticazione per la risorsa di Servizi cognitivi e la **posizione** in cui è distribuita. Salvare le modifiche.
4. Si noti che la cartella **speaking-clock** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: speaking-clock.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Speech SDK:

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. Nella funzione **Main** si noti che il codice per caricare la chiave e l'area di Servizi cognitivi dal file di configurazione è già stato fornito. È necessario usare queste variabili per creare un elemento **SpeechConfig** per la risorsa di Servizi cognitivi. Aggiungere il codice seguente sotto il commento **Configure speech service**:

    **C#**
    
    ```C#
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```
    
    **Python**
    
    ```Python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

6. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

7. Se si usa C#, è possibile ignorare eventuali avvisi relativi all'uso dell'operatore **await** nei metodi asincroni. Questo problema verrà risolto in un secondo momento. Il codice dovrebbe visualizzare l'area della risorsa del servizio Voce che verrà utilizzata dall'applicazione.

## <a name="recognize-speech"></a>Riconoscimento vocale

È ora disponibile un elemento **SpeechConfig** per il servizio Voce nella risorsa di Servizi cognitivi ed è quindi possibile usare l'API di **conversione della voce in testo scritto** per riconoscere il parlato e trascriverlo in testo.

### <a name="if-you-have-a-working-microphone"></a>Se è disponibile un microfono funzionante

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **TranscribeCommand** per accettare l'input vocale.
2. Nella funzione **TranscribeCommand**, sotto il commento **Configure speech recognition**, aggiungere il codice appropriato seguente per creare un client **SpeechRecognizer** che può essere usato per riconoscere e trascrivere il parlato usando il microfono di sistema predefinito:

    **C#**
    
    ```C#
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```
    
    **Python**
    
    ```Python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

3. Passare ora alla sezione **Aggiungere il codice per elaborare il comando trascritto** di seguito.

### <a name="alternatively-use-audio-input-from-a-file"></a>In alternativa, usare l'input audio da un file.

1. Nella finestra del terminale immettere il comando seguente per installare una libreria che può essere usata per riprodurre il file audio:

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

2. Nel file di codice per il programma aggiungere il codice seguente sotto le importazioni dello spazio dei nomi esistenti per importare la libreria appena installata:

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

3. Nella funzione **Main** si noti che il codice usa la funzione **TranscribeCommand** per accettare l'input vocale. Nella funzione **TranscribeCommand**, sotto il commento **Configure speech recognition**, aggiungere il codice appropriato seguente per creare un client **SpeechRecognizer** che può essere usato per riconoscere e trascrivere il parlato da un file audio:

    **C#**

    ```C#
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # Configure speech recognition
    audioFile = 'time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

### <a name="add-code-to-process-the-transcribed-command"></a>Aggiungere il codice per elaborare il comando trascritto

1. Nella funzione **TranscribeCommand**, sotto il commento **Process speech input**, aggiungere il codice seguente per l'ascolto dell'input vocale, prestando attenzione a non sostituire il codice alla fine della funzione che restituisce il comando:

    **C#**
    
    ```C#
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**
    
    ```Python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Se si usa un microfono, parlare chiaramente e pronunciare "che ora è?". Il programma trascriverà l'input vocale e visualizzerà l'ora in base all'ora locale del computer in cui è in esecuzione il codice, che potrebbe non essere l'ora corretta in cui ci si trova.

    SpeechRecognizer offre circa 5 secondi per parlare. Se non rileva alcun input vocale, produce un risultato di tipo "Nessuna corrispondenza".

    Se SpeechRecognizer rileva un errore, viene generato il risultato "Cancelled". Il codice nell'applicazione visualizza quindi il messaggio di errore. La causa più probabile è una chiave o un'area non corretta nel file di configurazione.

## <a name="synthesize-speech"></a>Sintesi vocale

L'applicazione dell'orologio vocale accetta l'input vocale, ma in realtà non parla. Per risolvere questo problema, aggiungere codice per sintetizzare il parlato.

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **TellTime** per indicare all'utente l'ora corrente.
2. Nella funzione **TellTime**, sotto il commento **Configure speech synthesis**, aggiungere il codice seguente per creare un client **SpeechSynthesizer** che può essere usato per generare l'output vocale:

    **C#**
    
    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```
    
    > **Nota**: *la configurazione audio predefinita usa il dispositivo audio di sistema predefinito per l'output, quindi non è necessario specificare in modo esplicito un elemento **AudioConfig**. Se è necessario reindirizzare l'output audio a un file, è possibile usare un elemento **AudioConfig** con un percorso file a tale scopo.*

3. Nella funzione **TellTime**, sotto il commento **Synthesize spoken output**, aggiungere il codice seguente per generare l'output vocale, prestando attenzione a non sostituire il codice alla fine della funzione che stampa la risposta:

    **C#**
    
    ```C#
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

4. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

5. Quando richiesto, parlare chiaramente nel microfono e dire "che ora è?". Il programma parlerà, indicando l'ora.

## <a name="use-a-different-voice"></a>Usare una voce diversa

L'applicazione dell'orologio vocale usa una voce predefinita, che è possibile modificare. Il servizio Voce supporta una gamma di voci *standard* e voci *neurali* più simili a quelle umane. È anche possibile creare voci *personalizzate*.

> **Nota**: per un elenco delle voci neurali e standard, vedere [Supporto di lingua e voce](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech) nella documentazione del servizio Voce.

1. Nella funzione **TellTime**, sotto il commento **Configure speech synthesis**, modificare il codice come segue per specificare una voce alternativa prima di creare il client **SpeechSynthesizer**:

   **C#**

    ```C#
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural"; // add this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```
    
    **Python**
    
    ```Python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-RyanNeural' # add this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Quando richiesto, parlare chiaramente nel microfono e dire "che ora è?". Il programma parlerà con la voce specificata, indicando l'ora.

## <a name="use-speech-synthesis-markup-language"></a>Usare Speech Synthesis Markup Language

Speech Synthesis Markup Language (SSML) consente di personalizzare la modalità di sintesi vocale usando un formato basato su XML.

1. Nella funzione **TellTime** sostituire tutto il codice corrente sotto il commento **Synthesize spoken output** con il codice seguente (lasciare il codice sotto il commento **Print the response**):

   **C#**

    ```C#
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**
    
    ```Python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **speaking-clock**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python speaking-clock.py
    ```

3. Quando richiesto, parlare chiaramente nel microfono e dire "che ora è?". Il programma parlerà nella voce specificata in SSML (eseguendo l'override della voce specificata in SpeechConfig), indicando l'ora e quindi, dopo una pausa, indicando che è il momento di terminare il lab, effettivamente.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'uso delle API di **conversione della voce in testo scritto** e **sintesi vocale**, vedere la [documentazione sulla conversione della voce in testo scritto](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-to-text) e la [documentazione sulla sintesi vocale](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-text-to-speech).

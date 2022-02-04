---
lab:
  title: Tradurre il parlato
  module: Module 4 - Building Speech-Enabled Applications
ms.openlocfilehash: 45c8d0d31bee5901247b22917d8dbad8c587a0bd
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 01/12/2022
ms.locfileid: "135801353"
---
# <a name="translate-speech"></a>Tradurre il parlato

Il servizio **Voce** include un'API **Traduzione vocale** che consente di tradurre il linguaggio parlato. Si supponga ad esempio di voler sviluppare un'applicazione di traduzione che possa essere usata dagli utenti quando viaggiano in luoghi di cui non conoscono la lingua locale. Gli utenti potrebbero pronunciare frasi quali "Dove si trova la stazione?" o "Ho bisogno di trovare una farmacia" nella propria lingua e ottenere la traduzione nella lingua locale.

**Nota**: per questo esercizio è necessario usare un computer con altoparlanti/cuffia. Per un'esperienza ottimale è necessario anche un microfono. Alcuni ambienti virtuali ospitati potrebbero essere in grado di acquisire audio dal microfono locale, ma se questo approccio non funziona o non è disponibile alcun microfono, è possibile usare un file audio fornito per l'input vocale. Seguire con attenzione le istruzioni, poiché sarà necessario scegliere opzioni diverse in base alla scelta di usare un microfono oppure il file audio.

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
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva sarà necessario usare una delle chiavi e la posizione in cui è stato effettuato il provisioning del servizio, disponibili in questa pagina.

## <a name="prepare-to-use-the-speech-translation-service"></a>Preparazione all'uso del servizio Traduzione vocale

In questo esercizio verrà completata un'applicazione client parzialmente implementata che usa Speech SDK per riconoscere, tradurre e sintetizzare il parlato.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per la lingua preferita.

1. Nel riquadro **Explorer** di Visual Studio Code passare alla cartella **08-speech-translation** ed espandere la cartella **C-Sharp** o **Python** in base alla scelta della lingua.
2. Fare clic con il pulsante destro del mouse sulla cartella **translator** e aprire un terminale integrato. Installare quindi il pacchetto Speech SDK eseguendo il comando appropriato per la lingua scelta:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.19.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.19.0
    ```

3. Visualizzare i contenuti della cartella **translator** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori di configurazione inclusi nel file in modo che includano una **chiave** di autenticazione per la risorsa di Servizi cognitivi e la **posizione** in cui è distribuita. Salvare le modifiche.
4. Si noti che la cartella **translator** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: translator.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti allo spazio dei nomi esistenti, trovare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Speech SDK:

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. Nella funzione **Main** si noti che il codice per caricare la chiave e l'area di Servizi cognitivi dal file di configurazione è già stato fornito. È necessario usare queste variabili per creare un **speechTranslationConfig** per la risorsa di Servizi cognitivi, che verrà usato per tradurre l'input vocale. Aggiungere il codice seguente sotto il commento **Configure translation**:

    **C#**
    
    ```C#
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. **SpeechTranslationConfig** verrà usato per tradurre il parlato in testo, ma verrà usato anche **SpeechConfig** per sintetizzare le traduzioni in parlato. Aggiungere il codice seguente sotto il commento **Configure translation**:

    **C#**
    
    ```C#
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. Salvare le modifiche e tornare al terminale integrato per la cartella **translator**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. Se si usa C#, è possibile ignorare eventuali avvisi relativi all'uso dell'operatore **await** nei metodi asincroni. Questo problema verrà risolto in un secondo momento. Il codice dovrebbe mostrare un messaggio che indica che è pronto per tradurre da en-US. Premere INVIO per terminare il programma.

## <a name="implement-speech-translation"></a>Implementare la traduzione vocale

È ora disponibile un **SpeechTranslationConfig** per il servizio Voce nella risorsa di Servizi cognitivi ed è quindi possibile usare l'API **Traduzione vocale** per riconoscere e tradurre il parlato.

### <a name="if-you-have-a-working-microphone"></a>Se è disponibile un microfono funzionante

1. Nella funzione **Main** per il programma si noti che il codice usa la funzione **Translate** per tradurre l'input vocale.
2. Nella funzione **Translate** sotto il commento **Translate speech** aggiungere il codice seguente per creare un client **TranslationRecognizer** che può essere usato per riconoscere e tradurre il parlato usando il microfono di sistema predefinito per l'input.

    **C#**
    
    ```C#
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Nota**: il codice nell'applicazione converte l'input in tutte e tre le lingue in un'unica chiamata. Viene visualizzata solo la traduzione per la lingua specifica, ma è possibile recuperare qualsiasi traduzione specificando il codice della lingua di destinazione nella raccolta **translations** del risultato.

3. Passare ora alla sezione **Eseguire il programma** disponibile più avanti.

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

3. Nella funzione **Main** per il programma si noti che il codice usa la funzione **Translate** per tradurre l'input vocale. Nella funzione **Translate** sotto il commento **Translate speech** aggiungere quindi il codice seguente per creare un client **TranslationRecognizer** che può essere usato per riconoscere e tradurre il parlato da un file.

    **C#**
    
    ```C#
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **Nota**: il codice nell'applicazione converte l'input in tutte e tre le lingue in un'unica chiamata. Viene visualizzata solo la traduzione per la lingua specifica, ma è possibile recuperare qualsiasi traduzione specificando il codice della lingua di destinazione nella raccolta **translations** del risultato.

### <a name="run-the-program"></a>Eseguire il programma

1. Salvare le modifiche e tornare al terminale integrato per la cartella **translator**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

2. Quando richiesto, immettere un codice della lingua valido (*fr*, *es* o *hi*) e quindi, se si usa un microfono, parlare in modo chiaro e pronunciare la frase "Dove si trova la stazione?" o un'altra frase che potrebbe risultare utile quando si viaggia all'estero. Il programma dovrebbe trascrivere l'input vocale e tradurlo nella lingua specificata (francese, spagnolo o hindi). Ripetere questo processo, provando ogni lingua supportata dall'applicazione. Al termine, premere INVIO per terminare il programma.

    > **Nota**: TranslationRecognizer offre circa 5 secondi per parlare. Se non rileva alcun input vocale, produce un risultato di tipo "Nessuna corrispondenza".
    >
    > È possibile che la traduzione in hindi non venga sempre visualizzata correttamente nella finestra della console a causa di problemi di codifica dei caratteri.

## <a name="synthesize-the-translation-to-speech"></a>Sintetizzare la traduzione in parlato

Fino a questo punto, l'applicazione traduce l'input vocale in testo, che potrebbe risultare sufficiente se è necessario richiedere assistenza a qualcuno durante un viaggio. Sarebbe tuttavia meglio se la traduzione fosse pronunciata a voce alta con una voce appropriata.

1. Nella funzione **Translate** sotto il commento **Synthesize translation** aggiungere il codice seguente per usare un client **SpeechSynthesizer** per sintetizzare la traduzione come parlato tramite l'altoparlante predefinito.

    **C#**
    
    ```C#
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. Salvare le modifiche e tornare al terminale integrato per la cartella **translator**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. Quanto richiesto, immettere un codice della lingua valido (*fr*, *es* o *hi*) e quindi parlare in modo chiaro nel microfono e pronunciare una frase che potrebbe risultare utile quando si viaggia all'estero. Il programma dovrebbe trascrivere l'input vocale e rispondere con una traduzione vocale. Ripetere questo processo, provando ogni lingua supportata dall'applicazione. Al termine, premere INVIO per terminare il programma.

    > **Nota** *In questo esempio è stato usato **SpeechTranslationConfig** per tradurre parlato in testo e quindi è stato usato **SpeechConfig** per sintetizzare la traduzione come parlato. È infatti possibile usare **SpeechTranslationConfig** per sintetizzare direttamente la traduzione, ma questo approccio funziona solo quando si traduce in una singola lingua e restituisce un flusso audio che viene in genere salvato come file invece che inviato direttamente a un parlante.*

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'uso dell'API **Traduzione vocale**, vedere la [documentazione di Traduzione vocale](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation).

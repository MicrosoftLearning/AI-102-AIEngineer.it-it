---
lab:
  title: Creare un modello di comprensione del linguaggio con il servizio Lingua (anteprima)
ms.openlocfilehash: 92d556e50c8f5c827e16f1e5ad301d3b59c1ad6e
ms.sourcegitcommit: cb2edc850cec8390a81212d9b8f6a279d2f42b4d
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/04/2022
ms.locfileid: "141368843"
---
# <a name="create-a-language-understanding-model-with-the-language-service-preview"></a>Creare un modello di comprensione del linguaggio con il servizio Lingua (anteprima)

> **Nota**: la funzionalità di comprensione del linguaggio di conversazione del servizio Lingua è attualmente in anteprima ed è soggetta a modifiche. In alcuni casi, il training del modello può non riuscire. In tal caso, riprovare.  

Il servizio Lingua consente di definire un modello di *comprensione del linguaggio di conversazione* che le applicazioni possono usare per interpretare l'input in linguaggio naturale degli utenti, stimare la *finalità* degli utenti, ossia cosa vogliono ottenere, e identificare le *entità* a cui applicare la finalità.

Un modello linguistico per conversazioni per un'applicazione orologio potrebbe ad esempio elaborare un input simile al seguente:

*What is the time in London?*

Questo tipo di input è un esempio di *espressione* (qualcosa che un utente potrebbe dire o digitare) la cui *finalità* è recuperare l'orario in una località specifica (un'*entità* ), in questo caso Londra.

> **Nota**: l'attività di un modello linguistico per conversazioni è stimare la finalità dell'utente e identificare le entità a cui si applica. <u>Non</u> è compito di un modello linguistico per conversazioni eseguire effettivamente le azioni necessarie per soddisfare la finalità. Ad esempio, un'applicazione orologio può usare un modello linguistico per conversazioni per comprendere che l'utente vuole conoscere l'ora di Londra, ma è l'applicazione client stessa che deve quindi implementare la logica per determinare l'ora corretta e presentarla all'utente.

## <a name="create-a-language-service-resource"></a>Creare una risorsa del servizio Lingua

Per creare un modello linguistico per conversazioni, è necessaria una risorsa del **servizio Lingua** in un'area supportata. Al momento della stesura di questo articolo, sono supportate solo le aree Stati Uniti occidentali 2 ed Europa occidentale.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *lingua* e creare una risorsa del **servizio Lingua** con le impostazioni seguenti.

    - **Funzionalità**: usare le funzionalità predefinite, inclusa la comprensione del linguaggio di conversazione (anteprima) 
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: Stati Uniti occidentali 2 o Europa occidentale
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: selezionare il livello Standard (S). La comprensione del linguaggio di conversazione non è attualmente supportata nel livello Gratuito.
    - **Note legali**: _accettare le note_ 
    - **Avviso per intelligenza artificiale responsabile**: _accettare_
3. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.

## <a name="create-a-conversational-language-understanding-project"></a>Creare un progetto di comprensione del linguaggio di conversazione

Ora che è stata creata una risorsa di creazione, è possibile usarla per creare un progetto di comprensione del linguaggio di conversazione.

1. In una nuova scheda del browser aprire il portale Language Studio all'indirizzo `https://language.cognitive.azure.com/` e accedere usando l'account Microsoft associato alla sottoscrizione di Azure.

2. Se viene richiesto di scegliere una risorsa del servizio Lingua, selezionare le impostazioni seguenti:

    - **Directory di Azure**: directory di Azure contenente la sottoscrizione.
    - **Sottoscrizione di Azure**: sottoscrizione di Azure in uso.
    - **Language resource** (Risorsa Lingua): risorsa del servizio Lingua creata in precedenza.

3. Se <u>non</u> viene chiesto di scegliere una risorsa del servizio Lingua, è possibile che sia già stata assegnata una diversa risorsa di questo tipo e in tal caso:

    1. Sulla barra nella parte superiore della pagina fare clic sul pulsante **Impostazioni (&#9881;)** .
    2. Nella pagina **Impostazioni** visualizzare la scheda **Risorse**.
    3. Selezionare la risorsa del servizio Lingua appena creata e fare clic su **Cambiare risorsa**.
    4. Nella parte superiore della pagina fare clic su **Language Studio** per tornare alla home page di Language Studio.

4. Nella parte superiore del portale scegliere **Comprensione del linguaggio di conversazione** dal menu **Crea nuovo**.

5. Nella finestra di dialogo **Create a project**, nella pagina **Enter basic information** immettere i dati seguenti e quindi fare clic su **Avanti**:
    - **Nome**: `Clock`
    - **Descrizione**: `Natural language clock`
    - **Utterances primary language** (Lingua principale espressioni): Inglese
    - **Enable multiple languages in project?** (Abilitare più lingue nel progetto?): *opzione deselezionata*

7. Nella pagina **Rivedere e completare** fare clic su **Crea**.

## <a name="create-intents"></a>Creare finalità

La prima cosa da fare nel nuovo progetto è definire alcune finalità.

> **Suggerimento**: quando si lavora al progetto, se vengono visualizzati alcuni suggerimenti, leggerli e fare clic su **OK** per eliminarli oppure fare clic su **Ignora tutto**.

1. Nella scheda **Finalità** della pagina **Build schema** (Crea schema) selezionare **&#65291; Aggiungi** per aggiungere una nuova finalità denominata **GetTime**.

2. Fare clic sulla nuova finalità **GetTime** per modificarla e aggiungere le espressioni seguenti come input utente di esempio:

    `what is the time?`

    `what's the time?`

    `what time is it?`

    `tell me the time`

3. Dopo avere aggiunto queste espressioni, fare clic su **Salva le modifiche** e tornare alla pagina **Build schema** (Crea schema).

4. Aggiungere un'altra nuova finalità denominata **GetDay** con le espressioni seguenti:

    `what day is it?`

    `what's the day?`

    `what is the day today?`

    `what day of the week is it?`

5. Dopo avere aggiunto le espressioni e averle salvate, tornare alla pagina **Build schema** (Crea schema) e aggiungere un'altra nuova finalità denominata **GetDate** con le espressioni seguenti:

    `what date is it?`

    `what's the date?`

    `what is the date today?`

    `what's today's date?`

6. Dopo avere aggiunto queste espressioni, salvarle e cancellare il filtro **GetDate** nella pagina delle espressioni in modo da poter visualizzare tutte le espressioni per tutte le finalità.

## <a name="train-and-test-the-model"></a>Eseguire il training e il test del modello

Ora che sono state aggiunte alcune finalità, è possibile eseguire il training del modello linguistico e verificare se riesce a stimarle correttamente dall'input dell'utente.

1. Nel riquadro a sinistra selezionare la pagina **Train model** e quindi selezionare **Start a training job**.

2. Nella finestra di dialogo **Start a training job** selezionare l'opzione per eseguire il training di un nuovo modello, denominarla **Clock** e assicurarsi che l'opzione esegua la valutazione quando il training è abilitato. 

3. Per iniziare il processo di training del modello, fare clic su **Train**.

4. Al termine del training (che può richiedere alcuni minuti) lo **stato** del processo cambierà in **Training succeeded**.

5. Selezionare la pagina **View model details** e quindi selezionare il modello **Clock**. Visualizzare le metriche di valutazione complessive e per finalità (*precisione*, *richiamo* e *punteggio F1*) e la *matrice di confusione* generata dalla valutazione eseguita durante il training (si noti che a causa del numero ridotto di espressioni di esempio, nei risultati potrebbero non essere incluse tutte le finalità).

    >**Nota**: per altre informazioni sulle metriche di valutazione, vedere la [documentazione](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

5. Nella pagina **Deploy model** selezionare **Add deployment**.

5. Nella finestra di dialogo **Add deployment** selezionare **Create a new deployment name** e quindi immettere **production**

5. Selezionare il modello **Clock** e fare clic su **Submit**. Questa distribuzione potrebbe richiedere alcuni minuti.

6. Dopo la distribuzione del modello, nella pagina **Testa modello** selezionare il modello **Clock**.

6. Nella pagina **Test Model: Clock**, nell'elenco a discesa **Deployment name** selezionare **production**.  

7. Immettere il testo seguente e quindi fare clic su **Esegui il test**:

    `what's the time now?`

    Esaminare il risultato restituito, notando che include la finalità prevista (che dovrebbe essere **GetTime**) e un punteggio di attendibilità che indica la probabilità calcolata dal modello per la finalità prevista. La scheda JSON mostra l'attendibilità comparativa per ogni finalità potenziale (quella con il punteggio di attendibilità più alto è la finalità stimata)

8. Deselezionare la casella di testo e quindi eseguire un altro test con il testo seguente:

    `tell me the time`

    Di nuovo, esaminare la finalità prevista e il punteggio di attendibilità.

9. Provare il testo seguente:

    `what's the day today?`

    Il modello dovrebbe prevedere la finalità **GetDay**.

## <a name="add-entities"></a>Aggiungere entità

Finora sono stati definite alcune espressioni semplici associate a finalità. La maggior parte delle applicazioni reali include espressioni più complesse da cui è necessario estrarre entità di dati specifiche per ottenere più contesto per la finalità.

### <a name="add-a-learned-entity"></a>Aggiungere un'entità di *apprendimento*

Il tipo più comune di entità è quella di *apprendimento*, in cui il modello impara a identificare i valori dell'entità in base agli esempi.

1. In Language Studio tornare alla pagina **Build schema** (Crea schema) e quindi nella scheda **Entità** selezionare **&#65291; Aggiungi** per aggiungere una nuova entità.

2. Nella finestra di dialogo **Add an entity** (Aggiungi un'entità) immettere il nome di entità **Location** e verificare che sia selezionata l'opzione **Learned** (Apprendimento). Fare quindi clic su **Aggiungi entità**.

3. Dopo aver creato l'entità **Location**, tornare alla pagina **Build schema** (Crea schema) e quindi nella scheda **Finalità** selezionare la finalità **GetTime**.

4. Immettere la nuova espressione di esempio seguente:

    `what time is it in London?`

5. Dopo avere aggiunto l'espressione, selezionare la parola ***London** _ e nell'elenco a discesa visualizzato selezionare _ *Location** per indicare che "London" è un esempio di località.

6. Aggiungere un'altra espressione di esempio:

    `Tell me the time in Paris?`

7. Dopo avere aggiunto l'espressione, selezionare la parola ***Paris** _ ed eseguirne il mapping all'entità _ *Location**.

8. Aggiungere un'altra espressione di esempio:

    `what's the time in New York?`

9. Dopo avere aggiunto l'espressione, selezionare le parole ***New York** _ ed eseguirne il mapping all'entità _ *Location**.

10. Fare clic su **Salva le modifiche** per salvare le nuove espressioni.

### <a name="add-a-list-entity"></a>Aggiungere un'entità *elenco*

In alcuni casi, i valori validi per un'entità possono essere limitati a un elenco di termini e sinonimi specifici. Questo consente all'app di identificare più facilmente le istanze dell'entità nelle espressioni.

1. In Language Studio tornare alla pagina **Build schema** (Crea schema) e quindi nella scheda **Entità** selezionare **&#65291; Aggiungi** per aggiungere una nuova entità.

2. Nella finestra di dialogo **Add an entity** (Aggiungi un'entità) immettere il nome di entità **Weekday** e selezionare il tipo di entità **Elenco**. Fare quindi clic su **Aggiungi entità**.

3. Nella pagina per l'entità **Weekday** fare clic su **&#65291; Add new list** nella sezione **List**. Immettere quindi il valore e il sinonimo seguenti e fare clic su **Salva**:

    | List key | sinonimi|
    |-------------------|---------|
    | Sunday | Dom |

4. Ripetere il passaggio precedente per aggiungere i componenti dell'elenco seguenti:

    | Valore | sinonimi|
    |-------------------|---------|
    | Monday | Mon |
    | Tuesday | Tue, Tues |
    | Wednesday | Wed, Weds |
    | Thursday | Thur, Thurs |
    | Friday | Fri |
    | Sabato | Sat |

5. Tornare alla pagina **Build schema** (Crea schema) e quindi nella scheda **Finalità** selezionare la finalità **GetDate**.

6. Immettere la nuova espressione di esempio seguente:

    `what date was it on Saturday?`

7. Dopo avere aggiunto l'espressione, selezionare la parola ***Saturday** _ e nell'elenco a discesa visualizzato selezionare _*Weekday**.

8. Aggiungere un'altra espressione di esempio:

    `what date will it be on Friday?`

9. Dopo avere aggiunto l'espressione, eseguire il mapping di **Friday** all'entità **Weekday**.

10. Aggiungere un'altra espressione di esempio:

    `what will the date be on Thurs?`

11. Dopo avere aggiunto l'espressione, eseguire il mapping di **Thurs** all'entità **Weekday**.

12. Fare clic su **Salva le modifiche** per salvare le nuove espressioni.

### <a name="add-a-prebuilt-entity"></a>Aggiungere un'entità *predefinita*

Il servizio Lingua fornisce un set di entità *predefinite* comunemente usate nelle applicazioni di conversazione.

1. In Language Studio tornare alla pagina **Build schema** (Crea schema) e quindi nella scheda **Entità** selezionare **&#65291; Aggiungi** per aggiungere una nuova entità.

2. Nella finestra di dialogo **Add an entity** immettere il nome di entità **Date** e selezionare il tipo di entità **Prebuilt**. Fare quindi clic su **Aggiungi entità**.

3. Nella sezione **Predefinito** della pagina per l'entità **Date** fare clic su **&#65291; Add new prebuilt** (Aggiungi nuovo elemento predefinito).

4. Nell'elenco **Select prebuilt** (Seleziona elemento predefinito) selezionare **Data/Ora** e quindi fare clic su **Salva**.

5. Tornare alla pagina **Build schema** (Crea schema) e quindi nella scheda **Finalità** selezionare la finalità **GetDay**.

6. Immettere la nuova espressione di esempio seguente:

    `what day was 01/01/1901?`

7. Dopo avere aggiunto l'espressione, selezionare ***01/01/1901** _ e nell'elenco a discesa visualizzato selezionare _*Date**.

8. Aggiungere un'altra espressione di esempio:

    `what day will it be on Dec 31st 2099?`

9. Dopo avere aggiunto l'espressione, eseguire il mapping di **Dec 31st 2099** all'entità **Date**.

10. Fare clic su **Salva le modifiche** per salvare le nuove espressioni.

### <a name="retrain-the-model"></a>Ripetere il training del modello

Ora che è stato modificato lo schema, è necessario ripetere il training e il test del modello.

1. Nella pagina **Train model** selezionare **Start a training job**.

1. Nella pagina **Start a training job** selezionare l'opzione per sovrascrivere il modello esistente e specificare il modello **Clock**. Assicurarsi che sia selezionata l'opzione per eseguire la valutazione durante il training e fare clic su **Esegui il training** per eseguire il training del modello, confermando di voler sovrascrivere il modello esistente.

2. Al termine del training, lo **stato** del processo verrà aggiornato in **Training succeeded**. 

2. Selezionare la pagina **View model details** e quindi selezionare il modello **Clock**. Visualizzare le metriche di valutazione complessive, per entità e per finalità (*precisione*, *richiamo* e *punteggio F1*) e la *matrice di confusione* generata dalla valutazione eseguita durante il training (si noti che a causa del numero ridotto di espressioni di esempio, nei risultati potrebbero non essere incluse tutte le finalità).

3. Nella pagina **Deploy model** selezionare **Add deployment**.

3. Nella finestra di dialogo **Add deployment** selezionare **Override an existing deployment name** e quindi selezionare **production**.

3. Selezionare il modello **Clock** e quindi fare clic su **Submit** per distribuirlo. Questa operazione potrebbe richiedere tempo.

4. Quando il modello viene distribuito, nella pagina **Test model** selezionare il modello **Clock**, selezionare la distribuzione **production** e quindi testarlo con il testo seguente:

    `what's the time in Edinburgh?`

5. Esaminare il risultato restituito, che dovrebbe stimare la finalità **GetTime** e un'entità **Location** con il valore di testo "Edinburgh".

6. Provare a testare le espressioni seguenti:

    `what time is it in Tokyo?`
    
    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## <a name="use-the-model-from-a-client-app"></a>Usare il modello da un'app client

In un progetto reale si perfezionerebbero in modo iterativo le finalità e le entità, ripetendo il training e i test finché non si è soddisfatti delle prestazioni di previsione. Dopo avere eseguito il test, se le prestazioni di stima sono soddisfacenti è possibile usare il modello in un'app client chiamando la relativa interfaccia REST. In questo esercizio si userà l'utilità *curl* per chiamare l'endpoint REST per il modello.

1. In Language Studio, nella pagina **Distribuisci modello** selezionare il modello **Clock**. Fare quindi clic su **Get prediction URL** (Ottieni URL di stima).

2. Nella finestra di dialogo **Get prediction URL** (Ottieni URL di stima) si noti che l'URL per l'endpoint di stima viene visualizzato insieme a una richiesta di esempio, costituita da un comando **curl** che invia una richiesta HTTP POST all'endpoint, specificando la chiave per la risorsa Lingua nell'intestazione e includendo una query e una lingua nei dati della richiesta.

3. Copiare la richiesta di esempio e incollarla nell'editor di testo preferito, ad esempio Blocco note.

4. Sostituire i segnaposto seguenti:
    - **YOUR_QUERY_HERE**: *What's the time in Sydney*
    - **QUERY_LANGUAGE_HERE**: *EN*

    Il comando sarà simile al codice seguente:

    ```
    curl -X POST "https://some-name.cognitiveservices.azure.com/language/:analyze-conversations?projectName=Clock&deploymentName=production&api-version=2021-11-01-preview" -H "Ocp-Apim-Subscription-Key: 0ab1c23de4f56..."  -H "Apim-Request-Id: 9zy8x76wv5u43...." -H "Content-Type: application/json" -d "{\"verbose\":true,\"query\":\"What's the time in Sydney?\",\"language\":\"EN\"}"
    ```

5. Aprire un prompt dei comandi (Windows) o una shell Bash (Linux/Mac).

6. Copiare e incollare il comando curl modificato nell'interfaccia della riga di comando ed eseguirlo.

7. Visualizzare il codice JSON risultante, che deve includere la finalità e le entità stimate, come nell'esempio seguente:

    ```
    {"query":"What's the time in Sydney?","prediction":{"topIntent":"GetTime","projectKind":"conversation","intents":[{"category":"GetTime","confidenceScore":0.9998859},{"category":"GetDate","confidenceScore":9.8372206E-05},{"category":"GetDay","confidenceScore":1.5763446E-05}],"entities":[{"category":"Location","text":"Sydney","offset":19,"length":6,"confidenceScore":1}]}}
    ```

8. Esaminare la risposta JSON restituita dal modello per verificare che la finalità con il punteggio più elevato sia **GetTime**.

9. Modificare la query nel comando curl in `What's today's date?` e quindi eseguirla ed esaminare il codice JSON risultante.

10. Provare le query seguenti:

    `What day will Jan 1st 2050 be?`

    `What time is it in Glasgow?`

    `What date will next Monday be?`

## <a name="export-the-project"></a>Esportare il progetto

È possibile usare Language Studio per sviluppare e testare il modello di comprensione del linguaggio, ma in un processo di sviluppo software per DevOps è necessario mantenere una definizione del progetto con controllo del codice sorgente che si possa includere nelle pipeline di integrazione continua e recapito continuo (CI/CD). Sebbene sia *possibile* usare l'API REST Lingua negli script di codice per creare il modello ed eseguirne il training, un modo più semplice consiste nell'usare il portale per creare lo schema del modello e quindi esportarlo come file con estensione *JSON* che è possibile importare e sottoporre di nuovo a training in un'altra istanza del servizio Lingua. Questo approccio consente di sfruttare i vantaggi dell'interfaccia visiva di Language Studio in termini di produttività, mantenendo al tempo stesso la portabilità e la riproducibilità del modello.

1. Nella pagina **Projects** di Language Studio selezionare il progetto **Clock**. Non fare clic su **Clock**, ma selezionare l'icona a forma di cerchio per selezionare il progetto Clock.

2. Fare clic sul pulsante **&#x2913; Esporta**.

3. Salvare il file **Clock.json** generato nella posizione desiderata.

4. Aprire il file scaricato nell'editor di codice preferito, ad esempio Visual Studio Code, per esaminare la definizione JSON del progetto.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sull'uso del servizio **Lingua** per creare soluzioni di comprensione del linguaggio di conversazione, vedere la [documentazione del servizio Lingua](https://docs.microsoft.com/azure/cognitive-services/language-service/conversational-language-understanding/overview).

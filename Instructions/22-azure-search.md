---
lab:
  title: Creare una soluzione di Ricerca cognitiva di Azure
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 38d5d50ba7e906ee0842a076ec08ad1e92d10293
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625850"
---
# <a name="create-an-azure-cognitive-search-solution"></a>Creare una soluzione di Ricerca cognitiva di Azure

Tutte le organizzazioni si basano sulle informazioni per prendere decisioni, rispondere a domande e funzionare in modo efficiente. Il problema per la maggior parte delle organizzazioni non è costituito dalla mancanza di informazioni, ma dalla difficoltà nell'individuare ed estrarre le informazioni dalla quantità elevata di documenti, database e altre origini in cui vengono archiviate le informazioni.

Si supponga, ad esempio, che *Margie's Travel* sia un'agenzia di viaggi specializzata nell'organizzazione di viaggi in città di tutto il mondo. Nel corso del tempo la società ha raccolto una grande quantità di informazioni in documenti quali le brochure, oltre a recensioni degli alberghi inviate dai clienti. Questi dati sono una fonte preziosa di informazioni dettagliate per il personale dell'agenzia e i clienti durante la pianificazione dei viaggi, ma il volume dei dati può rendere difficile l'individuazione di informazioni rilevanti per rispondere a una domanda specifica del cliente.

Per risolvere il problema, Margie's Travel può usare Ricerca cognitiva di Azure per implementare una soluzione in cui i documenti vengono indicizzati e arricchiti mediante competenze cognitive basate sull'intelligenza artificiale che agevolano le ricerche.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se il repository di codice **AI-102-AIEngineer** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-azure-resources"></a>Creare le risorse di Azure

La soluzione che verrà creata per Margie's Travel richiede le risorse seguenti nella sottoscrizione di Azure:

- Una risorsa di **Ricerca cognitiva di Azure**, che gestirà l'indicizzazione e l'esecuzione di query.
- Una risorsa di **Servizi cognitivi**, che fornisce servizi di intelligenza artificiale per le competenze che la soluzione di ricerca può usare per arricchire i dati nell'origine dati con informazioni dettagliate generate dall'intelligenza artificiale.
- Un **account di archiviazione** con un contenitore BLOB in cui vengono archiviati i documenti in cui eseguire la ricerca.

> **Importante**: le risorse Ricerca cognitiva di Azure e Servizi cognitivi devono trovarsi nella stessa posizione.

### <a name="create-an-azure-cognitive-search-resource"></a>Creare una risorsa di Ricerca cognitiva di Azure

1. In un Web browser aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *ricerca* e creare una risorsa di **Ricerca cognitiva di Azure** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *creare un nuovo gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito.*
    - **Nome del servizio**: *immettere un nome univoco*
    - **Posizione**: *selezionare una posizione, tenendo presente che le risorse di Ricerca cognitiva di Azure e Servizi cognitivi devono trovarsi nella stessa posizione*
    - **Piano tariffario**: Basic

3. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
4. Esaminare la pagina **Panoramica** nel pannello della risorsa di Ricerca cognitiva di Azure nel portale di Azure. In questo caso è possibile usare un'interfaccia visiva per creare, testare, gestire e monitorare i vari componenti di una soluzione di ricerca, incluse origini dati, indici, indicizzatori e set di competenze.

### <a name="create-a-cognitive-services-resource"></a>Creare una risorsa per Servizi cognitivi

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Servizi cognitivi**. La soluzione di ricerca userà questa funzionalità per arricchire i dati nell'archivio dati con informazioni dettagliate generate dall'intelligenza artificiale.

1. Tornare alla home page del portale di Azure e selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Servizi cognitivi* e creare una risorsa di **Servizi cognitivi** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *lo stesso gruppo di risorse della risorsa di Ricerca cognitiva di Azure*
    - **Regione**: *la stessa posizione della risorsa di Ricerca cognitiva di Azure*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
2. Selezionare le caselle di controllo necessarie e creare la risorsa.
3. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.

### <a name="create-a-storage-account"></a>Creare un account di archiviazione

1. Tornare alla home page del portale di Azure e selezionare il pulsante **&#65291;Crea una risorsa**, cercare *account di archiviazione* e creare una risorsa di **Account di archiviazione** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: **lo stesso gruppo di risorse delle risorse di Ricerca cognitiva di Azure e Servizi cognitivi*
    - **Nome account di archiviazione**: *immettere un nome univoco*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Prestazioni**: standard
    - **Replica**: archiviazione con ridondanza locale
2. Attendere il completamento della distribuzione e quindi passare alla risorsa distribuita.
3. Nella pagina **Panoramica** prendere nota dell'**ID sottoscrizione**, che identifica la sottoscrizione in cui viene effettuato il provisioning dell'account di archiviazione.
4. Nella pagina **Chiavi di accesso** notare che sono state generate due chiavi per l'account di archiviazione. Selezionare quindi **Mostra chiavi** per visualizzarle.

    > **Suggerimento**: tenere aperto il pannello **Account di archiviazione**. Nella procedura successiva saranno necessari l'ID sottoscrizione e una delle chiavi.

## <a name="upload-documents-to-azure-storage"></a>Caricare documenti in Archiviazione di Azure

Ora che sono disponibili le risorse necessarie, è possibile caricare alcuni documenti nell'account di archiviazione di Azure.

1. Nel riquadro **Explorer** di Visual Studio Code espandere la cartella **22-create-a-search-solution** e selezionare **UploadDocs.cmd**.
2. Modificare il file batch per sostituire i segnaposto **YOUR_SUBSCRIPTION_ID**, **YOUR_AZURE_STORAGE_ACCOUNT_NAME** e **YOUR_AZURE_STORAGE_KEY** con l'ID sottoscrizione, il nome dell'account di archiviazione di Azure e i valori della chiave dell'account di archiviazione di Azure appropriati per l'account di archiviazione creato in precedenza.
3. Salvare le modifiche, quindi fare clic con il pulsante destro del mouse sulla cartella **22-create-a-search-solution** e aprire un terminale integrato.
4. Immettere il comando seguente per accedere alla sottoscrizione di Azure usando l'interfaccia della riga di comando di Azure.

    ```
    az login
    ```

Verrà aperta una scheda del Web browser in cui si richiede di accedere ad Azure. Accedere, quindi chiudere la scheda del browser e tornare a Visual Studio Code.

5. Immettere il comando seguente per eseguire il file batch. Viene creato un contenitore BLOB nell'account di archiviazione e vengono caricati i documenti nella cartella **data**.

    ```
    UploadDocs
    ```

## <a name="index-the-documents"></a>Indicizzare i documenti

Una volta caricati i documenti, è possibile creare una soluzione di ricerca indicizzandoli.

1. Nel portale di Azure passare alla risorsa di Ricerca cognitiva di Azure. Quindi, nella pagina **Panoramica** selezionare **Importa dati**.
2. Nella pagina **Definisci la connessione ai dati** selezionare **Archiviazione BLOB di Azure** nell'elenco **Origine dati**. Completare quindi i dettagli dell'archivio dati con i valori seguenti:
    - **Origine dati**: Archiviazione BLOB di Azure
    - **Nome origine dati**: margies-data
    - **Dati da estrarre**: contenuto e metadati
    - **Modalità di analisi**: impostazione predefinita
    - **Stringa di connessione**: *selezionare **Scegliere una connessione esistente**, selezionare quindi l'account di archiviazione e infine il contenitore **margies** creato dallo script UploadDocs.cmd*.
    - **Autenticazione con identità gestita**: non selezionato
    - **Nome contenitore**: margies
    - **Cartella BLOB**: *lasciare vuoto questo campo*
    - **Descrizione**: brochure e recensioni nel sito Web di Margie's Travel.
3. Procedere con il passaggio successivo: *Aggiungi competenze cognitive*.
4. nella sezione **Collega Servizi cognitivi** selezionare la risorsa di Servizi cognitivi.
5. Nella sezione **Aggiungi arricchimenti**:
    - Modificare il **Nome del set di competenze** in **margies-skillset**.
    - Selezionare l'opzione **Abilita OCR e unisci tutto il testo nel campo merged_content**.
    - Assicurarsi che il **Campo dei dati di origine** sia impostato su **merged_content**.
    - Lasciare il campo **Livello di granularità dell'arricchimento** come **Campo di origine**, che imposta l'intero contenuto del documento da indicizzare. Tuttavia, si noti che è possibile modificare questa impostazione per estrarre informazioni a livelli più granulari, ad esempio pagine o frasi.
    - Selezionare i campi arricchiti seguenti:

        | Competenza cognitiva | Parametro | Nome del campo |
        | --------------- | ---------- | ---------- |
        | Estrai nomi di località | | locations |
        | Estrarre le frasi chiave | | keyphrase |
        | Rileva lingua | | Linguaggio |
        | Genera tag dalle immagini | | imageTags |
        | Genera sottotitoli in altre lingue dalle immagini | | imageCaption |

6. Controllare le selezioni, perché può essere difficile modificarle in un secondo momento. Procedere quindi al passaggio successivo: *Personalizza indice di destinazione*.
7. Modificare il **Nome indice** in **margies-index**.
8. Assicurarsi che la **Chiave** sia impostata su **metadata_storage_path** e lasciare vuoti i campi **Nome dello strumento suggerimenti** e **Modalità di ricerca**.
9. Apportare le modifiche seguenti ai campi dell'indice, lasciando in tutti gli altri campi le impostazioni predefinite (**IMPORTANTE**: potrebbe essere necessario scorrere verso destra per visualizzare l'intera tabella).

    | Nome del campo | Recuperabile | Filtrabile | Sortable | Con facet | Ricercabile |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | nome_archiviazione_metadati | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | autore_metadati | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrase | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | Linguaggio | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | |

11. Controllare le selezioni, assicurandosi in particolare che per ogni campo siano selezionate le opzioni **Recuperabile**, **Filtrabile**, **Ordinabile**, **Con facet** e **Ricercabile** corrette, perché può essere difficile modificarle in un secondo momento. Procedere quindi al passaggio successivo: *Crea un indicizzatore*.
12. Modificare il **Nome indicizzatore** in **margies-indexer**.
13. Lasciare la **Pianificazione** impostata su **Una volta**.
14. Espandere le opzioni **Avanzate** e assicurarsi che l'opzione **Chiavi di codifica Base 64** sia selezionata. In genere, le chiavi di codifica rendono l'indice più efficiente.
15. Selezionare **Invia** per creare l'origine dati, il set di competenze, l'indice e l'indicizzatore. L'indicizzatore viene eseguito automaticamente ed esegue la pipeline di indicizzazione, che:
    1. Estrae il contenuto e i campi dei metadati del documento dall'origine dati.
    2. Esegue il set di competenze cognitive per generare campi arricchiti aggiuntivi.
    3. Esegue il mapping all'indice dei campi estratti.
16. Nella metà inferiore della pagina **Panoramica** della risorsa di Ricerca cognitiva di Azure, visualizzare la scheda **Indicizzatori**, che dovrebbe mostrare l'indicizzatore **margies-indexer** appena creato. Attendere alcuni minuti e fare clic su **&orarr; Aggiorna** finché lo **Stato** non indica l'esito positivo.

## <a name="search-the-index"></a>Eseguire ricerche nell'indice

Ora è possibile eseguire ricerche nell'indice disponibile.

1. Nella parte superiore pagina **Panoramica** della risorsa di Ricerca cognitiva di Azure selezionare **Esplora ricerche**.
2. Nella casella **Stringa di query** di Esplora ricerche immettere `*` (un solo asterisco) e quindi selezionare **Cerca**.

    Questa query recupera tutti i documenti nell'indice in formato JSON. Esaminare i risultati e prendere nota dei campi di ciascun documento, che includono contenuto, metadati e dati arricchiti dei documenti, estratti dalle competenze cognitive selezionate.

3. Modificare la stringa di query in `search=*&$count=true` e avviare la ricerca.

    Questa volta, all'inizio dei risultati è incluso un campo **@odata.count** che indica il numero di documenti restituiti dalla ricerca.

4. Provare la stringa di query seguente:

    ```
    search=*&$count=true&$select=metadata_storage_name,metadata_author,locations
    ```

    Questa volta i risultati includono solo il nome del file, l'autore e qualsiasi percorso indicato nel contenuto dei documenti. Il nome e l'autore del file sono indicati nei campi **metadata_storage_name** e **metadata_author**, estratti dal documento di origine. Il campo **locations** è stato generato da una competenza cognitiva.

5. Provare ora la stringa di query seguente:

    ```
    search="New York"&$count=true&$select=metadata_storage_name,keyphrases
    ```

    Questa ricerca trova i documenti che menzionano "New York" in uno dei campi ricercabili e restituisce il nome file e le keyphrase nel documento.

6. Provare a eseguire un'altra stringa di query:

    ```
    search="New York"&$count=true&$select=metadata_storage_name&$filter=metadata_author eq 'Reviewer'
    ```

    Questa query restituisce il nome file di tutti i documenti creati dal *Revisore* che menzionano "New York".

## <a name="explore-and-modify-definitions-of-search-components"></a>Esplorare e modificare le definizioni dei componenti della ricerca

I componenti della soluzione di ricerca si basano su definizioni JSON, che è possibile visualizzare e modificare nel portale di Azure.

Anche se è possibile usare il portale per creare e modificare soluzioni di ricerca, è spesso consigliabile definire gli oggetti di ricerca in JSON e usare l'interfaccia REST di Servizi cognitivi di Azure per crearli e modificarli.

### <a name="get-the-endpoint-and-key-for-your-azure-cognitive-search-resource"></a>Ottenere l'endpoint e la chiave per la risorsa di Ricerca cognitiva di Azure

1. Nel portale di Azure tornare alla pagina **Panoramica** della risorsa di Ricerca cognitiva di Azure. Nella sezione superiore della pagina trovare l'**URL** della risorsa (che ha un aspetto simile a **https://resource_name.search.windows.net** ) e copiarlo negli Appunti.
2. Nel riquadro Explorer di Visual Studio Code espandere la cartella **22-create-a-search-solution** e la relativa sottocartella **modify-search** e selezionare **modify-search.cmd** per aprirla. Questo file di script verrà usato per eseguire i comandi *cURL* che inviano JSON all'interfaccia REST di Servizi cognitivi di Azure.
3. In **modify-search.cmd** sostituire il segnaposto **YOUR_SEARCH_URL** con l'URL copiato negli Appunti.
4. Nel portale di Azure visualizzare la pagina **Chiavi** della risorsa di Ricerca cognitiva di Azure e copiare la **Chiave di amministrazione primaria** negli Appunti.
5. In Visual Studio Code sostituire il segnaposto **YOUR_ADMIN_KEY** con la chiave copiata negli Appunti.
6. Salvare le modifiche apportate a **modify-search.cmd**, ma non eseguire il comando.

### <a name="review-and-modify-the-skillset"></a>Esaminare e modificare il set di competenze

1. Nella cartella **modify-search** di Visual Studio Code aprire **skillset.json**. Viene visualizzata una definizione JSON per **margies-skillset**.
2. Nella parte superiore della definizione del set di competenze si noti l'oggetto **cognitiveServices**, che viene usato per connettere la risorsa di Servizi cognitivi al set di competenze.
3. Nel portale di Azure aprire la risorsa di Servizi cognitivi (<u>non</u> la risorsa di Ricerca cognitiva di Azure) e visualizzarne la pagina **Chiavi**. Copiare quindi la **Chiave 1** negli Appunti.
4. In Visual Studio Code, in **skillset.json** sostituire il segnaposto **YOUR_COGNITIVE_SERVICES_KEY** con la chiave di Servizi cognitivi copiata negli Appunti.
5. Scorrere il file JSON, notando che include le definizioni delle competenze create usando l'interfaccia utente di Ricerca cognitiva di Azure nel portale di Azure. Nella parte inferiore dell'elenco delle competenze è stata inserita una competenza aggiuntiva con la definizione seguente:

    ```
    {
        "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "score",
                "targetName": "sentimentScore"
            }
        ]
    }
    ```

La nuova competenza è denominata **get-sentiment** e, per ogni livello **document** di un documento, valuterà il testo trovato nel campo **merged_content** del documento in fase di indicizzazione, che include il contenuto di origine e l'eventuale testo estratto dalle immagini nel contenuto. Usa la **lingua** estratta del documento (l'impostazione predefinita è l'inglese) e stima un punteggio per il sentiment del contenuto. Il punteggio viene quindi restituito come nuovo campo denominato **sentimentScore**.

6. Salvare le modifiche apportate a **skillset.json**.

### <a name="review-and-modify-the-index"></a>Esaminare e modificare l'indice

1. Nella cartella **modify-search** di Visual Studio Code aprire **index.json**. Viene visualizzata una definizione JSON per **margies-index**.
2. Scorrere l'indice e visualizzare le definizioni dei campi. Alcuni campi si basano su metadati e contenuto del documento di origine, mentre altri sono il risultato di competenze nel set di competenze.
3. Al termine dell'elenco di campi definiti nel portale di Azure sono stati inseriti due campi aggiuntivi:

    ```
    {
        "name": "sentiment",
        "type": "Edm.Double",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. Il campo **sentiment** verrà usato per aggiungere l'output della competenza **get-sentiment** che è stata aggiunta al set di competenze. Il campo **url** verrà usato per aggiungere all'indice l'URL di ogni documento indicizzato, in base al valore **metadata_storage_path** estratto dall'origine dati. Si noti che l'indice include già il campo **metadata_storage_path**, ma viene usato come chiave di indice con codifica Base 64, che lo rende efficiente come chiave ma richiede la decodifica da parte delle applicazioni client per usare il valore dell'URL effettivo come campo. L'aggiunta di un secondo campo per il valore non codificato risolve questo problema.

### <a name="review-and-modify-the-indexer"></a>Esaminare e modificare l'indicizzatore

1. Nella cartella **modify-search** di Visual Studio Code aprire **indexer.json**. Viene visualizzata una definizione JSON per **margies-indexer**, che esegue il mapping dei campi estratti dal contenuto e dai metadati del documento nella sezione **fieldMappings** e dei valori estratti dalle competenze nel set di competenze nella sezione **outputFieldMappings** ai campi nell'indice.
3. Nell'elenco **fieldMappings** prendere nota del mapping del valore **metadata_storage_path** al campo della chiave con codifica Base 64. Questo è stato creato durante l'assegnazione di **metadata_storage_path** come chiave, con l'opzione di codifica della chiave selezionata nel portale di Azure. Inoltre, è presente un nuovo mapping esplicito dello stesso valore al campo **url**, ma senza la codifica Base 64:

    ```
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }
    
    ```

Per tutti gli altri campi di metadati e contenuto nel documento di origine viene eseguito il mapping implicito a campi con lo stesso nome nell'indice.

4. Esaminare la sezione **ouputFieldMappings** che esegue il mapping degli output delle competenze nel set di competenze ai campi dell'indice. La maggior parte di queste opzioni riflette le scelte effettuate nell'interfaccia utente, ma è stato aggiunto il mapping seguente dal valore **sentimentScore** estratto dalla competenza sentiment al campo **sentiment** aggiunto all'indice:

    ```
    {
        "sourceFieldName": "/document/sentimentScore",
        "targetFieldName": "sentiment"
    }
    ```

### <a name="use-the-rest-api-to-update-the-search-solution"></a>Usare l'API REST per aggiornare la soluzione di ricerca

1. Fare clic con il pulsante destro del mouse sulla cartella **modify-search** e aprire un terminale integrato.
2. Nel riquadro del terminale per la cartella **modify-search** immettere il comando seguente per eseguire lo script **modify-search.cmd**, che invia le definizioni JSON all'interfaccia REST e avvia l'indicizzazione.

    ```
    modify-search
    ```

3. Al termine dell'esecuzione dello script, tornare alla pagina **Panoramica** della risorsa di Ricerca cognitiva di Azure nel portale di Azure e visualizzare la pagina **Indicizzatori**. Selezionare periodicamente **Aggiorna** per tenere traccia dello stato di avanzamento dell'operazione di indicizzazione. Il completamento dell'operazione potrebbe richiedere circa un minuto.

    *Potrebbero essere presenti degli avvisi per documenti troppo grandi per la valutazione del sentiment. Spesso l'analisi del sentiment viene eseguita a livello di pagina o di frase anziché a livello dell'intero documento. In questo caso, tuttavia, la maggior parte dei documenti, in particolare le recensioni di hotel, è abbastanza breve ed è possibile eseguire utili valutazioni dei punteggi del sentiment a livello di documento.*

### <a name="query-the-modified-index"></a>Eseguire query sull'indice modificato

1. Nella parte superiore del pannello della risorsa di Ricerca cognitiva di Azure selezionare **Esplora ricerche**.
2. Nella casella **Stringa di query** di Esplora ricerche immettere la stringa di query seguente e quindi selezionare **Cerca**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment gt 0.5
    ```

    Questa query recupera l'**URL**, il **sentiment** e le **keyphrase** per tutti i documenti che citano *Londra*, creati dal *revisore*, che hanno un punteggio del **sentiment** maggiore di *0,5*; in altre parole, recensioni positive che citano Londra.

3. Chiudere la pagina **Esplora ricerche** per tornare alla pagina **Panoramica**.

## <a name="create-a-search-client-application"></a>Creare un'applicazione client di ricerca

Ora che è disponibile un indice utile, è possibile usarlo da un'applicazione client. A tale scopo, è possibile usare l'interfaccia REST, inviare richieste e ricevere risposte in formato JSON tramite HTTP. In alternativa, è possibile usare l'SDK per il linguaggio di programmazione preferito. In questo esercizio si userà l'SDK.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

### <a name="get-the-endpoint-and-keys-for-your-search-resource"></a>Ottenere l'endpoint e le chiavi per la risorsa di ricerca

1. Nel portale di Azure, nella pagina **Panoramica** della risorsa di Ricerca cognitiva di Azure prendere nota del valore **url**, che dovrebbe essere simile a **https://*nome_della_risorsa*.search.windows.net**. Questo è l'endpoint della risorsa di ricerca.
2. Nella pagina **Chiavi** sono disponibili due chiavi **amministratore** e una singola chiave di **query**. Una chiave *amministratore* viene usata per creare e gestire le risorse di ricerca. Una chiave di *query* viene usata dalle applicazioni client che devono solo eseguire query di ricerca.

    *Saranno necessari l'endpoint e la chiave di query per l'applicazione client.*

### <a name="prepare-to-use-the-azure-cognitive-search-sdk"></a>Preparare l'uso dell'SDK di Ricerca cognitiva di Azure

1. Nel riquadro **Explorer** di Visual Studio Code passare alla cartella **22-create-a-search-solution** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **margies-travel** e aprire un terminale integrato. Installare quindi il pacchetto SDK di Ricerca cognitiva di Azure eseguendo il comando appropriato per il linguaggio scelto:

    **C#**
    
    ```
    dotnet add package Azure.Search.Documents --version 11.1.1
    ```
    
    **Python**
    
    ```
    pip install azure-search-documents==11.0.0
    ```
    
3. Visualizzare il contenuto della cartella **margies-travel** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e la **chiave di query** della risorsa di Ricerca cognitiva di Azure. Salvare le modifiche.

### <a name="explore-code-to-search-an-index"></a>Esplorare il codice per eseguire ricerche in un indice

La cartella **margies-travel** contiene i file di codice per un'applicazione Web (un'applicazione Web *ASP.NET Razor* in Microsoft C# o un'applicazione *Flask* in Python) che include funzionalità di ricerca.

1. Aprire il file di codice seguente nell'applicazione Web, a seconda del linguaggio di programmazione scelto:
    - **C#** :Pages/Index.cshtml.cs
    - **Python**: app.py
2. Nella parte superiore del file di codice trovare il commento **Import search namespaces** (Importa spazi dei nomi di ricerca) e prendere nota degli spazi dei nomi importati per l'uso con l'SDK di Ricerca cognitiva di Azure:
3. Nella funzione **search_query** trovare il commento **Create a search client** (Crea un client di ricerca) e notare che il codice crea un oggetto **SearchClient** usando l'endpoint e la chiave di query per la risorsa di Ricerca cognitiva di Azure:
4. Nella funzione **search_query** trovare il commento **Submit search query** (Invia query di ricerca) ed esaminare il codice per l'avvio di una ricerca del testo specificato con le opzioni seguenti:
    - Una *modalità di ricerca* che prevede che vengano trovate **tutte** le singole parole nel testo di ricerca.
    - Il numero totale di documenti trovati dalla ricerca è incluso nei risultati.
    - I risultati vengono filtrati in modo da includere solo i documenti che corrispondono all'espressione filtro specificata.
    - I risultati vengono presentati nell'ordine specificato.
    - Ciascun valore discreto del campo **metadata_author** viene restituito come *facet* che può essere usato per visualizzare valori predefiniti per il filtro.
    - Nei risultati sono inclusi fino a tre estratti dei campi **merged_content** e **imageCaption** con i termini di ricerca evidenziati.
    - I risultati includono solo i campi specificati.

### <a name="explore-code-to-render-search-results"></a>Esplorare il codice per eseguire il rendering dei risultati della ricerca

L'app Web include già il codice per elaborare ed eseguire il rendering dei risultati della ricerca.

1. Aprire il file di codice seguente nell'applicazione Web, a seconda del linguaggio di programmazione scelto:
    - **C#** :Pages/Index.cshtml
    - **Python**: templates/search.html
2. Esaminare il codice, che esegue il rendering della pagina in cui vengono visualizzati i risultati della ricerca. Si noti che:
    - La pagina inizia con un modulo di ricerca che l'utente può usare per inviare una nuova ricerca (nella versione Python dell'applicazione il modulo è definito nel modello **base.html**), a cui si fa riferimento all'inizio della pagina.
    - Viene quindi eseguito il rendering di un secondo modulo, consentendo all'utente di perfezionare i risultati della ricerca. Il codice per questo modulo:
        - Recupera e visualizza il conteggio dei documenti dai risultati della ricerca.
        - Recupera i valori dei facet per il campo **metadata_author** e li visualizza come elenco di opzioni per il filtro.
        - Crea un elenco a discesa di opzioni di ordinamento per i risultati.
    - Il codice esegue quindi l'iterazione dei risultati della ricerca, eseguendo il rendering di ogni risultato come segue:
        - Visualizza il campo **metadata_storage_name** (nome file) come collegamento all'indirizzo nel campo **url**.
        - Visualizza *evidenziazioni* per i termini di ricerca trovati nei campi **merged_content** e **imageCaption** per mostrare i termini di ricerca nel contesto.
        - Visualizza i campi **metadata_author**, **metadata_storage_size**, **metadata_storage_last_modified** e **language**.
        - Indica il **sentiment** usando un'emoticon (&#128578; per punteggi maggiori o uguali a 0,5 e &#128577; per punteggi minori di 0,5).
        - Visualizza le prime cinque **keyphrase** (se presenti).
        - Visualizza le prime cinque **posizioni** (se presenti).
        - Visualizza i primi cinque **imageTag** (se presenti).

### <a name="run-the-web-app"></a>Eseguire l'app Web

 1. Tornare al terminale integrato per la cartella **margies-travel**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. Nel messaggio visualizzato quando l'app viene avviata correttamente, seguire il collegamento all'applicazione Web ( *http://localhost:5000/* o *http://127.0.0.1:5000/* ) in esecuzione per aprire il sito di Margie's Travel in un Web browser.
3. Nel sito Web di Margie's Travel immettere **Hotel a Londra** nella casella di ricerca e fare clic su **Cerca**.
4. Esaminare i risultati della ricerca. Includono il nome del file, con un collegamento all'URL del file, un estratto del contenuto del file con i termini di ricerca (*Londra* e *hotel*) evidenziati e altri attributi del file dai campi dell'indice.
5. Si noti che la pagina dei risultati include alcuni elementi dell'interfaccia utente che consentono di affinare i risultati. Queste includono:
    - Un *filtro* basato su un valore del facet per il campo **metadata_author**. Questo dimostra come usare i campi *con facet* per restituire un elenco di *facet*: campi con un piccolo set di valori discreti che possono essere visualizzati come possibili valori del filtro nell'interfaccia utente.
    - La possibilità di *ordinare* i risultati in base a un campo e a una direzione di ordinamento specificati (crescente o decrescente). L'ordine predefinito si basa *pertinenza*, che viene calcolata come valore **search.score()** in base a un *profilo di punteggio* che valuta la frequenza e l'importanza dei termini di ricerca nei campi dell'indice.
6. Selezionare il filtro **Revisore** e l'opzione di ordinamento **Da positivo a negativo**, quindi **Affina risultati**.
7. Si noti che i risultati vengono filtrati in modo da includere solo le recensioni e vengono ordinati per sentiment decrescente.
8. Nella casella **Cerca** immettere una nuova ricerca di **hotel silenzioso a New York** ed esaminare i risultati.
9. Provare i termini di ricerca seguenti:
    - **Torre di Londra**, si noti che questo termine viene identificato come *keyphrase* in alcuni documenti.
    - **Grattacielo**, si noti che questa parola non è inclusa nel contenuto effettivo di alcun documento, ma si trova nelle *didascalie delle immagini* e nei *tag immagine* generati per le immagini in alcuni documenti.
    - **Deserto di Mojave**, si noti che questo termine viene identificato come *posizione* in alcuni documenti.
10. Chiudere la scheda del browser contenente il sito Web di Margie's Travel e tornare a Visual Studio Code. Quindi, nel terminale Python per la cartella **margies-travel** in cui è in esecuzione l'applicazione dotnet o flask, immettere CTRL+C per arrestare l'app.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni su Ricerca cognitiva di Azure, vedere la [documentazione di Ricerca cognitiva di Azure](https://docs.microsoft.com/azure/search/search-what-is-azure-search).

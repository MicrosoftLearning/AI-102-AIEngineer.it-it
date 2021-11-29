---
lab:
  title: Creare un archivio conoscenze con Ricerca cognitiva di Azure
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 8d7008ee592d4a76f24ca26a4b192c7dd5cbe9ea
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625816"
---
# <a name="create-a-knowledge-store-with-azure-cognitive-search"></a>Creare un archivio conoscenze con Ricerca cognitiva di Azure

Ricerca cognitiva di Azure usa una pipeline di arricchimento di competenze cognitive per estrarre dai documenti i campi generati dall'intelligenza artificiale e includerli in un indice di ricerca. Anche se l'indice può essere considerato l'output primario di un processo di indicizzazione, i dati arricchiti che contiene potrebbero essere utili anche in altri modi. Ad esempio:

- Poiché l'indice è essenzialmente una raccolta di oggetti JSON, ognuno dei quali rappresenta un record indicizzato, potrebbe essere utile esportare gli oggetti come file JSON per l'integrazione in un processo di orchestrazione dei dati usando strumenti come Azure Data Factory.
- Potrebbe essere necessario normalizzare i record dell'indice in uno schema relazionale di tabelle per l'analisi e la creazione di report con strumenti come Microsoft Power BI.
- Dopo aver estratto le immagini incorporate dai documenti durante il processo di indicizzazione, potrebbe essere necessario salvare tali immagini come file.

In questo esercizio si implementerà un archivio conoscenze per *Margie's Travel*, un'agenzia di viaggi fittizia che usa le informazioni contenute nelle brochure e nelle recensioni degli hotel per aiutare i clienti a pianificare i viaggi.

## <a name="clone-the-repository-for-this-course"></a>Clonare il repository per questo corso

Se è già stato clonato il repository di codice **AI-102-AIEngineer** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/AI-102-AIEngineer` in una cartella locale (non importa quale).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## <a name="create-azure-resources"></a>Creare le risorse di Azure

> **Nota**: se in precedenza è stato completato l'esercizio **[Creare una soluzione di Ricerca cognitiva di Azure](22-azure-search.md)** e le risorse di Azure sono ancora disponibili nella sottoscrizione, è possibile ignorare questa sezione e iniziare dalla sezione **Creare una soluzione di ricerca**. In caso contrario, seguire la procedura seguente per effettuare il provisioning delle risorse di Azure necessarie.

1. In un Web browser aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Visualizzare i **Gruppi di risorse** nella sottoscrizione.
3. Se si usa una sottoscrizione limitata in cui è stato fornito un gruppo di risorse, selezionare il gruppo di risorse per visualizzarne le proprietà. In caso contrario, creare un nuovo gruppo di risorse con il nome desiderato e passare a tale gruppo dopo che è stato creato.
4. Nella pagina **Panoramica** per il gruppo di risorse prendere nota dei valori di **ID sottoscrizione** e **Località**. Questi valori saranno necessari, insieme al nome del gruppo di risorse, nei passaggi successivi.
5. In Visual Studio Code espandere la cartella **24-knowledge-store** e selezionare **setup.cmd**. Questo script batch verrà usato per eseguire i comandi dell'interfaccia della riga di comando di Azure che consentono di creare le risorse di Azure necessarie.
6. Fare clic con il pulsante destro del mouse sulla cartella **24-knowledge-store** e scegliere **Apri nel terminale integrato**.
7. Nel riquadro del terminale immettere il comando seguente per stabilire una connessione autenticata alla sottoscrizione di Azure.

    ```
    az login --output none
    ```

8. Quando verrà richiesto, accedere alla sottoscrizione di Azure. Tornare quindi a Visual Studio Code e attendere il completamento del processo di accesso.
9. Eseguire il comando seguente per elencare le località di Azure.

    ```
    az account list-locations -o table
    ```

10. Nell'output individuare il valore **Name** corrispondente alla località del gruppo di risorse (ad esempio per *Stati Uniti orientali* il nome corrispondente è *eastus*).
11. Nello script **setup.cmd** modificare le dichiarazioni di variabili **subscription_id**, **resource_group** e **location** con i valori appropriati per ID sottoscrizione, nome del gruppo di risorse e nome della località in uso. Salvare quindi le modifiche.
12. Nel terminale per la cartella **24-knowledge-store** immettere il comando seguente per eseguire lo script:

    ```
    setup
    ```
    > **Nota**: il modulo dell'interfaccia della riga di comando di ricerca è in versione di anteprima e potrebbe bloccarsi con l'indicazione *- Running...* visualizzata. Se ciò accade per più di 2 minuti, premere CTRL+C per annullare l'operazione a esecuzione prolungata e quindi selezionare **N** quando viene chiesto se si vuole terminare lo script. L'operazione dovrebbe quindi venire completata correttamente.
    >
    > Se si verifica un errore dello script, assicurarsi di avere eseguito il salvataggio con i nomi delle variabili corretti e riprovare.

13. Al termine dello script, esaminare l'output visualizzato e prendere nota delle informazioni seguenti sulle risorse di Azure (questi valori saranno necessari in un secondo momento):
    - Nome account di archiviazione
    - Stringa di connessione di archiviazione
    - Account Servizi cognitivi
    - Chiave di Servizi cognitivi
    - Endpoint del servizio di ricerca
    - Chiave amministratore del servizio di ricerca
    - Chiave di query del servizio di ricerca

14. Nel portale di Azure aggiornare il gruppo di risorse e verificare che contenga l'account di archiviazione di Azure, la risorsa di Servizi cognitivi di Azure e la risorsa di Ricerca cognitiva di Azure.

## <a name="create-a-search-solution"></a>Creare una soluzione di ricerca

Ora che le risorse di Azure necessarie sono disponibili, è possibile creare una soluzione di ricerca costituita dai componenti seguenti:

- Un'**origine dati** che fa riferimento ai documenti nel contenitore di archiviazione di Azure.
- Un **set di competenze** che definisce una pipeline di arricchimento delle competenze per estrarre dai documenti i campi generati dall'intelligenza artificiale. Il set di competenze definisce anche le *proiezioni* che verranno generate nell'*archivio conoscenze*.
- Un **indice** che definisce un set di record di documenti in cui è possibile eseguire ricerche.
- Un **indicizzatore** che estrae i documenti dall'origine dati, applica il set di competenze e popola l'indice. Il processo di indicizzazione salva inoltre in modo permanente le proiezioni definite nel set di competenze nell'archivio conoscenze.

In questo esercizio si userà l'interfaccia REST di Ricerca cognitiva di Azure per creare questi componenti inviando richieste JSON.

### <a name="prepare-json-for-rest-operations"></a>Preparare JSON per le operazioni REST

Si userà l'interfaccia REST per inviare definizioni JSON per i componenti di Ricerca cognitiva di Azure.

1. In Visual Studio Code, nella cartella **24-knowledge-store** espandere la cartella **create-search** e selezionare **data_source.json**. Questo file contiene una definizione JSON per un'origine dati denominata **margies-knowledge-data**.
2. Sostituire il segnaposto **YOUR_CONNECTION_STRING** con la stringa di connessione per l'account di archiviazione di Azure, che sarà simile alla seguente:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *È possibile trovare la stringa di connessione nella pagina **Chiavi di accesso** per l'account di archiviazione nel portale di Azure.*

3. Salvare e chiudere il file JSON aggiornato.
4. Nella cartella **create-search** aprire **skillset.json**. Questo file contiene una definizione JSON per un set di competenze denominato **margies-knowledge-data**.
5. Nella parte superiore della definizione del set di competenze, nell'elemento **cognitiveServices** sostituire il segnaposto **YOUR_COGNITIVE_SERVICES_KEY** con una delle chiavi per le risorse di Servizi cognitivi.

    *È possibile trovare le chiavi nella pagina **Chiavi ed endpoint** della risorsa di Servizi cognitivi nel portale di Azure.*

6. Alla fine della raccolta di competenze nel set di competenze, individuare la competenza **Microsoft.Skills.Util.ShaperSkill** denominata **define-projection**. Questa competenza definisce una struttura JSON per i dati arricchiti che verranno usati per le proiezioni che la pipeline salverà in modo permanente nell'archivio conoscenze per ogni documento elaborato dall'indicizzatore.
7. Nella parte inferiore del file del set di competenze osservare che il set di competenze include anche una definizione **knowledgeStore**, che include a sua volta una stringa di connessione per l'account di archiviazione di Azure in cui deve essere creato l'archivio conoscenze e una raccolta di **proiezioni**. Questo set di competenze include tre *gruppi di proiezioni*:
    - Un gruppo contenente una proiezione di *oggetto* basata sull'output **knowledge_projection** della competenza shaper nel set di competenze.
    - Un gruppo contenente una proiezione di *file* basata sulla raccolta **normalized_images** di dati di immagini estratti dai documenti.
    - Un gruppo contenente le proiezioni di *tabelle* seguenti:
        - **KeyPhrases**: contiene una colonna chiave generata automaticamente e una colonna **keyPhrase** mappata all'output della raccolta **knowledge_projection/key_phrases/** della competenza shaper.
        - **Locations**: contiene una colonna chiave generata automaticamente e una colonna **location** mappata all'output della raccolta **knowledge_projection/key_phrases/** della competenza shaper.
        - **ImageTags**: contiene una colonna chiave generata automaticamente e una colonna **tag** mappata all'output della raccolta **knowledge_projection/image_tags/** della competenza shaper.
        - **Docs**: contiene una colonna chiave generata automaticamente e tutti i valori di output di **knowledge_projection** della competenza shaper non già assegnati a una tabella.
8. Sostituire il segnaposto **YOUR_CONNECTION_STRING** per il valore **storageConnectionString** con la stringa di connessione per l'account di archiviazione.
9. Salvare e chiudere il file JSON aggiornato.
10. Nella cartella **create-search** aprire **index.json**. Questo file contiene una definizione JSON per un indice denominato **margies-knowledge-index**.
11. Esaminare il codice JSON per l'indice e quindi chiudere il file senza apportare modifiche.
12. Nella cartella **create-search** aprire **indexer.json**. Questo file contiene una definizione JSON per un indicizzatore denominato **margies-knowledge-indexer**.
13. Esaminare il codice JSON per l'indicizzatore e quindi chiudere il file senza apportare modifiche.

### <a name="submit-rest-requests"></a>Inviare richieste REST

Dopo avere preparato gli oggetti JSON che definiscono i componenti della soluzione di ricerca, è possibile inviare i documenti JSON all'interfaccia REST per crearli.

1. Nella cartella **create-search** aprire **create-search.cmd**. Questo script batch usa l'utilità cURL per inviare le definizioni JSON all'interfaccia REST per la risorsa di Ricerca cognitiva di Azure.
2. Sostituire le variabili segnaposto **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** con l'**URL** e una delle **chiavi amministratore** della risorsa di Ricerca cognitiva di Azure.

    *È possibile trovare questi valori nelle pagine **Panoramica** e **Chiavi** per la risorsa di Ricerca cognitiva di Azure nel portale di Azure.*

3. Salvare il file batch aggiornato.
4. Fare clic con il pulsante destro del mouse sulla cartella **create-search** e scegliere **Apri nel terminale integrato**.
5. Nel riquadro del terminale per la cartella **create-search** immettere il comando seguente per eseguire lo script batch.

    ```
    create-search
    ```

6. Al termine dello script, nella pagina della risorsa di Ricerca cognitiva di Azure nel portale di Azure selezionare la pagina **Indicizzatori** e attendere il completamento del processo di indicizzazione.

    *È possibile selezionare **Aggiorna** per tenere traccia dello stato di avanzamento dell'operazione di indicizzazione. Il completamento può richiedere circa un minuto.*

    > **Suggerimento**: in caso di errore dello script, controllare i segnaposto aggiunti nei file **data_source.json** e **skillset.json**, nonché nel file **create-search.cmd**. Dopo aver corretto eventuali errori, può essere necessario usare l'interfaccia utente del portale di Azure per eliminare tutti i componenti creati nella risorsa di ricerca prima di eseguire nuovamente lo script.

## <a name="view-the-knowledge-store"></a>Visualizzare l'archivio conoscenze

Dopo aver eseguito un indicizzatore che usa un set di competenze per creare un archivio conoscenze, i dati arricchiti estratti dal processo di indicizzazione vengono salvati in modo permanente nelle proiezioni dell'archivio conoscenze.

### <a name="view-object-projections"></a>Visualizzare le proiezioni di oggetti

Le proiezioni di *oggetti* definite nel set di competenze di Margie's Travel sono costituite da un file JSON per ogni documento indicizzato. Questi file vengono archiviati in un contenitore BLOB nell'account di archiviazione di Azure specificato nella definizione del set di competenze.

1. Nel portale di Azure visualizzare l'account di archiviazione di Azure creato in precedenza.
2. Selezionare la scheda **Storage Explorer** (nel riquadro a sinistra) per visualizzare l'account di archiviazione nell'interfaccia dello strumento di esplorazione dell'archiviazione nel portale di Azure.
2. Espandere **CONTENITORI BLOB** per visualizzare i contenitori nell'account di archiviazione. Oltre al contenitore **margies** in cui sono archiviati i dati di origine, saranno presenti due nuovi contenitori: **margies-images** e **margies-knowledge**. Questi contenitori sono stati creati dal processo di indicizzazione.
3. Selezionare il contenitore **margies-knowledge**. Tale contenitore include una cartella per ogni documento indicizzato.
4. Aprire una delle cartelle, quindi scaricare e aprire il file **knowledge-projection.json** in essa contenuto. Ogni file JSON contiene una rappresentazione di un documento indicizzato, inclusi i dati arricchiti estratti dal set di competenze, come illustrato di seguito.

```
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

La possibilità di creare proiezioni di *oggetti* di questo tipo consente di generare oggetti dati arricchiti che possono essere incorporati in una soluzione di analisi dei dati aziendali, ad esempio tramite l'inserimento dei file JSON in una pipeline di Azure Data Factory per un'ulteriore elaborazione o il caricamento in un data warehouse.

### <a name="view-file-projections"></a>Visualizzare le proiezioni di file

Le proiezioni di *file* definite nel set di competenze creano file JPEG per ogni immagine estratta dai documenti durante il processo di indicizzazione.

1. Nell'interfaccia dello strumento di esplorazione dell'archiviazione nel portale di Azure selezionare il contenitore BLOB **margies-images**. Questo contenitore include una cartella per ogni documento contenente immagini.
2. Aprire una delle cartelle e visualizzarne il contenuto. Ogni cartella contiene almeno un file \*.jpg.
3. Aprire uno dei file di immagine per verificare che siano presenti le immagini estratte dai documenti.

La possibilità di generare proiezioni di *file* di questo tipo rende l'indicizzazione un modo efficiente per estrarre immagini incorporate da un volume elevato di documenti.

### <a name="view-table-projections"></a>Visualizzare le proiezioni di tabelle

Le proiezioni di *tabelle* definite nel set di competenze formano uno schema relazionale di dati arricchiti.

1. Nell'interfaccia dello strumento di esplorazione dell'archiviazione nel portale di Azure espandere **TABELLE**.
2. Selezionare la tabella **Docs** per visualizzare le relative colonne. Le colonne includono alcune colonne di tabella standard di Archiviazione di Azure. Per nasconderle, modificare l'impostazione **Opzioni colonne** in modo da selezionare solo le colonne seguenti:
    - **document_id** (colonna chiave generata automaticamente dal processo di indicizzazione)
    - **file_id** (URL del file codificato)
    - **file_name** (nome del file estratto dai metadati del documento)
    - **language** (lingua in cui è scritto il documento)
    - **sentiment** (punteggio del sentiment calcolato per il documento)
    - **url** (URL del BLOB del documento in Archiviazione di Azure)
3. Visualizzare le altre tabelle create dal processo di indicizzazione:
    - **ImageTags** (contiene una riga per ogni singolo tag di immagine con il valore di **document_id** per il documento in cui è presente il tag).
    - **KeyPhrases** (contiene una riga per ogni singola frase chiave con il valore di **document_id** per il documento in cui è presente la frase).
    - **Locations** (contiene una riga per ogni singola località con il valore di **document_id** per il documento in cui è presente la località).

La possibilità di creare proiezioni di *tabelle* consente di compilare soluzioni analitiche e di creazione di report per l'esecuzione di query sullo schema relazionale, ad esempio usando Microsoft Power BI. Le colonne chiave generate automaticamente possono essere usate per unire le tabelle nelle query, ad esempio per restituire tutte le località indicate in un documento specifico.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla creazione di archivi conoscenze con Ricerca cognitiva di Azure, vedere la [documentazione di Ricerca cognitiva di Azure](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro).

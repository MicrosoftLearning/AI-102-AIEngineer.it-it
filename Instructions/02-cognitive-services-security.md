---
lab:
  title: Gestire la sicurezza di Servizi cognitivi
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: 9c8de44265ffa0846b6860fd7d416bb3be547ed9
ms.sourcegitcommit: 5ffc20f6a590fe643c2b695b8dc04589411be36e
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 05/31/2022
ms.locfileid: "145951183"
---
# <a name="manage-cognitive-services-security"></a>Gestire la sicurezza di Servizi cognitivi

La sicurezza è una considerazione fondamentale per qualsiasi applicazione e, in quanto sviluppatore, è necessario assicurarsi che l'accesso a risorse come i servizi cognitivi sia limitato solo a coloro che lo richiedono.

L'accesso ai servizi cognitivi viene in genere controllato tramite chiavi di autenticazione, che vengono generate quando si crea inizialmente una risorsa di Servizi cognitivi.

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
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, è possibile che non si sia autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0
3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.

## <a name="manage-authentication-keys"></a>Gestire le chiavi di autenticazione

Quando è stata creata la risorsa di Servizi cognitivi, sono state generate due chiavi di autenticazione. È possibile gestirle nel portale di Azure o tramite l'interfaccia della riga di comando di Azure.

1. Nel portale di Azure passare alla risorsa di Servizi cognitivi e visualizzare la relativa pagina **Chiavi ed endpoint**. Questa pagina contiene le informazioni necessarie per connettersi alla risorsa e usarla dalle applicazioni sviluppate. In particolare:
    - Un *endpoint* HTTP a cui le applicazioni client possono inviare richieste.
    - Due *chiavi* che possono essere usate per l'autenticazione (le applicazioni client possono usare entrambe le chiavi. Una pratica comune consiste nell'usarne una per lo sviluppo e un'altra per la produzione. È possibile rigenerare facilmente la chiave di sviluppo dopo che gli sviluppatori hanno terminato il lavoro per impedire l'accesso continuo).
    - La *località* in cui è ospitata la risorsa. Questa informazione è necessaria per le richieste ad alcune API, ma non a tutte.
2. In Visual Studio Code fare clic con il pulsante destro del mouse sulla cartella **02-cognitive-security** e aprire un terminale integrato. Immettere quindi il comando seguente per accedere alla sottoscrizione di Azure usando l'interfaccia della riga di comando di Azure.

    ```
    az login
    ```

    Verrà aperta una scheda del Web browser in cui si richiede di accedere ad Azure. Accedere, quindi chiudere la scheda del browser e tornare a Visual Studio Code.

    > **Suggerimento**: se si hanno più sottoscrizioni, è necessario assicurarsi di usare quella che contiene la risorsa di Servizi cognitivi.  Usare questo comando per determinare la sottoscrizione corrente. L'ID univoco è il valore **ID** nel codice JSON restituito.

    > **Avviso:** se si verifica un errore di verifica del certificato per `az login`, provare ad attendere qualche minuto e riprovare.
    >
    > ```
    > az account show
    > ```
    >
    > Se è necessario modificare la sottoscrizione, eseguire questo comando, modificando *&lt;Your_Subscription_Id&gt;* con l'ID sottoscrizione corretto.
    >
    > ```
    > az account set --subscription <Your_Subscription_Id>
    > ```
    >
    > In alternativa, è possibile specificare in modo esplicito l'ID sottoscrizione come parametro *--subscription* in ogni comando dell'interfaccia della riga di comando di Azure seguente.

3. È ora possibile usare il comando seguente per ottenere l'elenco delle chiavi di Servizi cognitivi, sostituendo *&lt;resourceName&gt;* con il nome della risorsa di Servizi cognitivi e *&lt;resourceGroup&gt;* con il nome del gruppo di risorse in cui è stata creata.

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

Il comando restituisce un elenco delle chiavi per la risorsa di Servizi cognitivi. Sono presenti due chiavi denominate **key1** e **key2**.

4. Per testare il servizio cognitivo, è possibile usare **curl**, uno strumento da riga di comando per le richieste HTTP. Nella cartella **02-cognitive-security** aprire **rest-test.cmd** e modificare il comando **curl** in esso contenuto (illustrato di seguito), sostituendo *&lt;yourEndpoint&gt;* e *&lt;yourKey&gt;* con l'URI dell'endpoint e la chiave **Key1** per usare l'API Analisi del testo nella risorsa di Servizi cognitivi.

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.0/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':[{'id':1,'text':'hello'}]}"
    ```

5. Salvare le modifiche e quindi nel terminale integrato per la cartella **02-cognitive-security** eseguire il comando seguente:

    ```
    rest-test
    ```

Il comando restituisce un documento JSON contenente informazioni sulla lingua rilevata nei dati di input (che sarà l'inglese).

6. Se una chiave viene compromessa o gli sviluppatori che la possiedono non richiedono più l'accesso, è possibile rigenerarla nel portale o tramite l'interfaccia della riga di comando di Azure. Eseguire il comando seguente per rigenerare la **chiave key1** (sostituendo *&lt;resourceName&gt;* e *&lt;resourceGroup&gt;* per la risorsa).

    ```
    az cognitiveservices account keys regenerate --name <resourceName> --resource-group <resourceGroup> --key-name key1
    ```

Viene restituito l'elenco delle chiavi per la risorsa di Servizi cognitivi. Si noti che **key1** è cambiato dall'ultimo recupero.

7. Eseguire nuovamente il comando **rest-test** con la vecchia chiave (è possibile usare il tasto **^** per scorrere i comandi precedenti) e verificare che ora abbia esito negativo.
8. Modificare il comando *curl* in **rest-test.cmd** sostituendo la chiave con il nuovo valore **key1** e salvare le modifiche. Eseguire quindi di nuovo il comando **rest-test** e verificare che abbia esito positivo.

> **Suggerimento:** in questo esercizio sono stati usati i nomi completi dei parametri dell'interfaccia della riga di comando di Azure, ad esempio **--resource-group**.  È anche possibile usare alternative più brevi, ad esempio **-g**, per rendere i comandi meno dettagliati (ma un po' più difficili da comprendere).  Il [Riferimento ai comandi dell'interfaccia della riga di comando di Servizi cognitivi](https://docs.microsoft.com/cli/azure/cognitiveservices?view=azure-cli-latest) elenca le opzioni dei parametri per ogni comando dell'interfaccia della riga di comando di Servizi cognitivi.

## <a name="secure-key-access-with-azure-key-vault"></a>Proteggere l'accesso alla chiave con Azure Key Vault

È possibile sviluppare applicazioni che utilizzano Servizi cognitivi usando una chiave per l'autenticazione. Ciò significa tuttavia che il codice dell'applicazione deve essere in grado di ottenere la chiave. Un'opzione consiste nell'archiviare la chiave in una variabile di ambiente o in un file di configurazione in cui viene distribuita l'applicazione, ma questo approccio lascia la chiave vulnerabile agli accessi non autorizzati. Un approccio migliore quando si sviluppano applicazioni in Azure consiste nell'archiviare la chiave in modo sicuro in Azure Key Vault e fornire l'accesso alla chiave tramite un'*identità gestita* (in altre parole, un account utente usato dall'applicazione stessa).

### <a name="create-a-key-vault-and-add-a-secret"></a>Creare un insieme di credenziali delle chiavi e aggiungere un segreto

Prima di tutto, è necessario creare un insieme di credenziali delle chiavi e aggiungere un *segreto* per la chiave di Servizi cognitivi.

1. Prendere nota del valore **key1** per la risorsa di Servizi cognitivi o copiarlo negli Appunti.
2. Nella pagina **Home** del portale di Azure selezionare il pulsante **&#65291;Crea una risorsa**, cercare *Key Vault* e creare una risorsa **Key Vault** con le impostazioni seguenti:
    - **Sottoscrizione**: *la propria sottoscrizione di Azure*
    - **Gruppo di risorse**: *lo stesso gruppo di risorse della risorsa di Servizi cognitivi*
    - **Nome dell'insieme di credenziali delle chiavi**: *immettere un nome univoco*
    - **Area**: *la stessa area della risorsa di Servizi cognitivi*
    - **Piano tariffario**: Standard
3. Attendere il completamento della distribuzione e quindi passare alla risorsa dell'insieme di credenziali delle chiavi.
4. Nel pannello di navigazione a sinistra selezionare **Segreti** (nella sezione Impostazioni).
5. Selezionare **+ Genera/Importa** e aggiungere un nuovo segreto con le impostazioni seguenti:
    - **Opzioni di caricamento**: Manuale
    - **Nome**: Cognitive-Services-Key *(è importante che corrisponda esattamente, perché in seguito si eseguirà il codice che recupera il segreto in base a questo nome)*
    - **Valore**: *chiave di Servizi cognitivi **key1***

### <a name="create-a-service-principal"></a>Creare un'entità servizio

Per accedere al segreto nell'insieme di credenziali delle chiavi, l'applicazione deve usare un'entità servizio che abbia accesso al segreto. Si userà l'interfaccia della riga di comando di Azure per creare l'entità servizio, individuare il relativo ID oggetto e concedere l'accesso al segreto in Azure Vault.

1. Tornare a Visual Studio Code e nel terminale integrato per la cartella **02-cognitive-security** eseguire il comando dell'interfaccia della riga di comando di Azure seguente, sostituendo *&lt;spName&gt;* con un nome appropriato per un'identità dell'applicazione, ad esempio *ai-app*. Sostituire anche *&lt;subscriptionId&gt;* e *&lt;resourceGroup&gt;* con i valori corretti per l'ID sottoscrizione e il gruppo di risorse contenente le risorse di Servizi cognitivi e dell'insieme di credenziali delle chiavi:

    > **Suggerimento**: se non si è certi dell'ID sottoscrizione, usare il comando **az account show** per recuperare le informazioni sulla sottoscrizione. L'ID sottoscrizione è l'attributo **id** nell'output.

    ```
    az ad sp create-for-rbac -n "api://<spName>" --role owner --scopes subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
    ```

L'output di questo comando include informazioni sulla nuova entità servizio. L'aspetto sarà simile al seguente:

    ```
    {
        "appId": "abcd12345efghi67890jklmn",
        "displayName": "ai-app",
        "name": "http://ai-app",
        "password": "1a2b3c4d5e6f7g8h9i0j",
        "tenant": "1234abcd5678fghi90jklm"
    }
    ```

Prendere nota dei valori **appId**, **password** e **tenant**, in quanto saranno necessari in un secondo momento (se si chiude questo terminale, non sarà possibile recuperare la password. È quindi importante prendere nota dei valori. È possibile incollare l'output in un nuovo file di testo in Visual Studio Code per assicurarsi di poter trovare i valori necessari in un secondo momento).

2. Per ottenere l'**ID oggetto** dell'entità servizio, eseguire il comando seguente dell'interfaccia della riga di comando di Azure, sostituendo *&lt;appId&gt;* con il valore dell'ID app dell'entità servizio.

    ```
    az ad sp show --id <appId> --query objectId --out tsv
    ```

3. Per assegnare alla nuova entità servizio l'autorizzazione per accedere ai segreti in Key Vault, eseguire il comando seguente dell'interfaccia della riga di comando di Azure, sostituendo *&lt;keyVaultName&gt;* con il nome della risorsa di Azure Key Vault e *&lt;objectId&gt;* con il valore dell'ID oggetto dell'entità servizio.

    ```
    az keyvault set-policy -n <keyVaultName> --object-id <objectId> --secret-permissions get list
    ```

### <a name="use-the-service-principal-in-an-application"></a>Usare l'entità servizio in un'applicazione

A questo punto è possibile usare l'identità dell'entità servizio in un'applicazione, in modo che possa accedere alla chiave segreta di Servizi cognitivi nell'insieme di credenziali delle chiavi e usarla per connettersi alla risorsa di Servizi cognitivi.

> **Nota**: in questo esercizio le credenziali dell'entità servizio verranno archiviate nella configurazione dell'applicazione e verranno usate per autenticare un'identità **ClientSecretCredential** nel codice dell'applicazione. Questa operazione è valida per lo sviluppo e il test, ma in un'applicazione di produzione reale un amministratore assegna un'*identità gestita* all'applicazione in modo che usi l'identità dell'entità servizio per accedere alle risorse, senza memorizzare nella cache o archiviare la password.

1. In Visual Studio Code espandere la cartella **02-cognitive-security** e la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **keyvault-client** e aprire un terminale integrato. Installare quindi i pacchetti necessari per usare Azure Key Vault e l'API Analisi del testo nella risorsa di Servizi cognitivi eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.1.0
    dotnet add package Azure.Identity --version 1.5.0
    dotnet add package Azure.Security.KeyVault.Secrets --version 4.2.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-textanalytics==5.1.0
    pip install azure-identity==1.5.0
    pip install azure-keyvault-secrets==4.2.0
    ```

3. Visualizzare il contenuto della cartella **keyvault-client** e notare che include un file per le impostazioni di configurazione:
    - **C#** : appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori di configurazione contenuti in base alle impostazioni seguenti:
    
    - **Endpoint** per la risorsa di Servizi cognitivi
    - Nome della risorsa di **Azure Key Vault**
    - **Tenant** dell'entità servizio
    - **ID applicazione** dell'entità servizio
    - **Password** dell'entità servizio

     Salvare le modifiche.
4. Si noti che la cartella **keyvault-client** contiene un file di codice per l'applicazione client:

    - **C#** : Program.cs
    - **Python**: keyvault-client.py

    Aprire il file di codice ed esaminare il codice in esso contenuto, notando i dettagli seguenti:
    - Lo spazio dei nomi per l'SDK installato viene importato
    - Il codice nella funzione **Main** recupera le impostazioni di configurazione dell'applicazione e quindi usa le credenziali dell'entità servizio per ottenere la chiave di Servizi cognitivi dall'insieme di credenziali delle chiavi.
    - La funzione **GetLanguage** usa l'SDK per creare un client per il servizio e quindi usa il client per rilevare la lingua del testo immesso.
5. Tornare nel terminale integrato per la cartella **keyvault-client**, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python keyvault-client.py
    ```

6. Quando richiesto, immettere del testo ed esaminare la lingua rilevata dal servizio. Ad esempio, provare a immettere "Hello", "Bonjour" e "Gracias".
7. Al termine del test dell'applicazione, immettere "quit" per arrestare il programma.

## <a name="more-information"></a>Altre informazioni

Per altre informazioni sulla protezione di Servizi cognitivi, vedere la [documentazione sulla sicurezza di Servizi cognitivi](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-security).
